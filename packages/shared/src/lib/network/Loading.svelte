<script lang="ts" generics="A">
	import LoadingState from '$lib/network/LoadingState.svelte';
	import type { Loadable } from '$lib/network/types';
	import type { Snippet } from 'svelte';

	type Props<A> = {
		loadable?: Loadable<A>;
		children: Snippet<[A]>;
	};

	const { loadable, children }: Props<A> = $props();
</script>

{#if loadable === undefined}
	<span>Uninitialized...</span>
{:else if loadable.status === 'found'}
	{@render children(loadable.value)}
{:else if loadable.status === 'loading'}
	<LoadingState />
{:else if loadable.status === 'not-found'}
	<span>Not found</span>
{:else if loadable.status === 'error'}
	<span>{loadable.error.name}</span>
	<span>{loadable.error.message}</span>
{:else}
	<span>Unknown state</span>
{/if}
