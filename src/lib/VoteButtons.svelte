<style>
	button {
		color: var(--sx-gray-200);
	}
	.vote-up:enabled:hover {
		color: var(--sx-pink-500);
	}
	.vote-up[aria-pressed='true'] {
		color: var(--sx-pink-400);
	}
	.vote-down:enabled:hover {
		color: var(--sx-blue-500);
	}
	.vote-down[aria-pressed='true'] {
		color: var(--sx-blue-400);
	}
	.vote-counter {
		height: 1.4rem;
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>

<div class={dir === 'row' ? 'f-row' : 'f-column'}>
	<button
		aria-pressed={votedUp}
		class="vote-up"
		class:small
		on:click={() => voteAs(1)}
		disabled={!loggedIn || votePending}
	>
		<span class="sr-only">Vote up</span>
		<Icon icon="arrow-up" variant="icon-only" />
	</button>
	<span class="vote-counter sx-badge-{counterColor} text-align-center align-self-center">
		{#if votePending}
			<Spinner />
		{:else}
			{score}
			<span class="sr-only">score</span>
		{/if}
	</span>
	<button
		aria-pressed={votedDown}
		class="vote-down"
		class:small
		on:click={() => voteAs(-1)}
		disabled={!loggedIn || votePending}
	>
		<span class="sr-only">Vote up</span>
		<Icon icon="arrow-down" variant="icon-only" />
	</button>
</div>

<script lang="ts">
	import { Icon } from 'sheodox-ui';
	import { createEventDispatcher } from 'svelte';
	import { getAppContext } from './app-context';
	import Spinner from './Spinner.svelte';

	const dispatch = createEventDispatcher<{
		vote: number;
	}>();

	const { loggedIn } = getAppContext();

	export let vote = 0;
	export let score = 0;
	export let small = false;
	export let dir: 'row' | 'column' = 'row';
	export let votePending = false;

	$: votedUp = vote === 1;
	// no vote is 0
	$: votedDown = vote === -1;

	$: counterColor = {
		'-1': 'blue',
		'0': 'gray',
		'1': 'pink'
	}[vote ?? 0];

	function voteAs(clickedVote: number) {
		const newVote = vote === clickedVote ? 0 : clickedVote;
		dispatch('vote', newVote);
	}
</script>
