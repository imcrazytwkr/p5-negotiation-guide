<script lang="ts">
	import { blur } from 'svelte/transition';
	import QuestionsList from '#lib/components/QuestionsList.svelte';
	import type { Question } from '#lib/types.ts';

	let { questions }: { questions: readonly Question[] } = $props();

	let searchValue = $state('');
	let opened = $state<Question | null>(null);
	let suggestedOpened = $state<Question | null>(null);

	const sortedQuestions = $derived(
		[...questions].sort((a, b) => {
			if (a.question < b.question) {
				return -1;
			} else if (a.question > b.question) {
				return 1;
			} else {
				return 0;
			}
		})
	);

	const normalizeString = (input: string): string => {
		return input.toLowerCase().replace(/[.,'/#!$%^&*;:{}=\-_`~()]/g, '');
	};

	const normalizedSearchValue = $derived(normalizeString(searchValue));

	const matchingQuestions = $derived(
		sortedQuestions.filter((question) =>
			opened
				? opened.uid === question.uid
				: normalizeString(question.chats[1]).includes(normalizedSearchValue)
		)
	);

	const matchingQuestionsToShow = $derived(matchingQuestions.length < 20 ? matchingQuestions : []);

	const showSuggested = $derived(opened !== null && opened.question === 0);

	const suggestedQuestions = $derived.by(() => {
		const current = opened;
		if (!current) return [];

		return questions.filter((question) =>
			suggestedOpened
				? suggestedOpened.uid === question.uid
				: question.question === 1 && question.type === current.type
		);
	});

	const onSearchInput = () => {
		opened = null;
		suggestedOpened = null;
	};
</script>

<input
	class="mb-12 w-full max-w-xl rounded-md bg-gray-800 px-4 py-2 outline-0 transition focus:-translate-y-0.5 focus:bg-gray-700 focus:shadow-md"
	placeholder="Search questions"
	bind:value={searchValue}
	oninput={onSearchInput}
/>

<QuestionsList
	questions={matchingQuestionsToShow}
	bind:opened
	onquestionClicked={() => (suggestedOpened = null)}
/>

{#if showSuggested}
	<h2 class="mb-4 text-xl" transition:blur>Suggested</h2>

	<QuestionsList questions={suggestedQuestions} bind:opened={suggestedOpened} transition={blur} />
{/if}
