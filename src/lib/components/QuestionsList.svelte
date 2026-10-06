<script lang="ts">
	import TwemojiAngerSymbol from '#lib/icons/TwemojiAngerSymbol.svg';
	import TwemojiSweatDroplets from '#lib/icons/TwemojiSweatDroplets.svg';
	import TwemojiSparkles from '#lib/icons/TwemojiSparkles.svg';
	import { blur, slide } from 'svelte/transition';
	import type { Question } from '#lib/types.ts';

	let {
		questions = [],
		opened = $bindable(null),
		transition,
		onquestionClicked
	}: {
		questions?: readonly Question[];
		opened?: Question | null;
		transition?: typeof blur;
		onquestionClicked?: (question: Question) => void;
	} = $props();

	const clickedQuestion = (question: Question) => {
		if (opened && opened.uid === question.uid) {
			opened = null;
		} else {
			opened = question;
		}

		onquestionClicked?.(question);
	};

	const reactionToEmoji = {
		good: TwemojiSparkles,
		ok: TwemojiSweatDroplets,
		bad: TwemojiAngerSymbol
	};
</script>

{#snippet list()}
	{#each questions as question (question.uid)}
		<div class="mb-4 w-full rounded-md bg-gray-800" transition:slide>
			<button
				class="w-full px-4 py-2 font-semibold text-balance"
				onclick={() => clickedQuestion(question)}
			>
				{question.chats[1]}
			</button>
			{#if opened?.uid === question.uid}
				<div transition:slide class="p-2">
					<table class="w-full">
						<thead>
							<tr>
								<th class="px-2"></th>
								<th class="px-2 text-fuchsia-500 after:content-['Ti'] sm:after:content-['Timid']"
								></th>
								<th class="px-2 text-red-500 after:content-['Ir'] sm:after:content-['Irritable']"
								></th>
								<th class="px-2 text-amber-500 after:content-['Up'] sm:after:content-['Upbeat']"
								></th>
								<th class="px-2 text-blue-500 after:content-['Gl'] sm:after:content-['Gloomy']"
								></th>
							</tr>
						</thead>
						<tbody>
							{#each question.choices as choice, i (i)}
								<tr>
									<td class="font-bold text-gray-300">
										{choice}
									</td>
									{#each question.reactions_table[i] as response, column (column)}
										<td class="px-2">
											<div class="flex h-5 w-full min-w-5">
												<img src={reactionToEmoji[response]} class="mx-auto" alt={response} />
											</div>
										</td>
									{/each}
								</tr>
							{/each}
						</tbody>
					</table>
				</div>
			{/if}
		</div>
	{/each}
{/snippet}

{#if transition}
	<div class="w-full max-w-xl" transition:transition>
		{@render list()}
	</div>
{:else}
	<div class="w-full max-w-xl">
		{@render list()}
	</div>
{/if}
