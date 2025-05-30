<script lang="ts">
	import BranchFilesList from '$components/BranchFilesList.svelte';
	import {
		createCommitStore,
		createIntegratedCommitsContextStore,
		createLocalCommitsContextStore,
		createLocalAndRemoteCommitsContextStore
	} from '$lib/commits/contexts';
	import { FileIdSelection } from '$lib/selection/fileIdSelection';
	import { getContext } from '@gitbutler/shared/context';
	import type { PatchSeries } from '$lib/branches/branch';
	import type { DetailedCommit } from '$lib/commits/commit';
	import type { RemoteFile } from '$lib/files/file';
	import type { LocalFile } from '$lib/files/file';
	import type { Writable } from 'svelte/store';

	interface Props {
		projectId?: string;
		files: LocalFile[] | RemoteFile[];
		isUnapplied: boolean;
		showCheckboxes?: boolean;
		commitDialogExpanded: Writable<boolean>;
		focusCommitDialog: () => void;
		allowMultiple?: boolean;
		readonly?: boolean;
		branches?: PatchSeries[];
	}

	const {
		projectId,
		files,
		isUnapplied,
		showCheckboxes = false,
		commitDialogExpanded,
		focusCommitDialog,
		allowMultiple = false,
		readonly = false,
		branches
	}: Props = $props();

	createCommitStore(undefined);
	const localCommits = createLocalCommitsContextStore([]);
	const localAndRemoteCommits = createLocalAndRemoteCommitsContextStore([]);
	const integratedCommits = createIntegratedCommitsContextStore([]);

	$effect(() => {
		if (branches) {
			const upstreamPatches: DetailedCommit[] = [];
			const localPatches: DetailedCommit[] = [];

			for (const branch of branches) {
				localPatches.push(...branch.patches);
				upstreamPatches.push(...branch.upstreamPatches);
			}

			localCommits.set(localPatches);
			localAndRemoteCommits.set(upstreamPatches);
			integratedCommits.set(localPatches.filter((p) => p.status === 'Integrated'));
		}
	});

	const fileIdSelection = getContext(FileIdSelection);
</script>

<div class="branch-files" role="presentation" onclick={() => fileIdSelection.clear()}>
	{#if files.length > 0}
		<BranchFilesList
			{projectId}
			{allowMultiple}
			{readonly}
			{files}
			{showCheckboxes}
			{isUnapplied}
			{commitDialogExpanded}
			{focusCommitDialog}
		/>
	{/if}
</div>

<style lang="postcss">
	.branch-files {
		flex: 1;
	}

	.branch-files:focus {
		outline: none;
	}
</style>
