<Tooltip {placement}>
	<span slot="tooltip">{text}</span>
	<button on:click class:small class={cl} disabled={disabled || busy}>
		{#if busy}
			<Spinner />
		{:else}
			<Icon {icon} variant="icon-only" />
			<span class="sr-only">{text}</span>
		{/if}
		<slot />
	</button>
</Tooltip>

<script lang="ts">
	import { Tooltip, Icon } from 'sheodox-ui';
	import Spinner from './Spinner.svelte';
	import type { Placement } from '@floating-ui/dom';

	export let small = false;
	export let disabled = false;
	export let busy = false;
	export let text = '';
	export let cl = '';
	export let placement: Placement | undefined = undefined;
	export let icon: string;
</script>
