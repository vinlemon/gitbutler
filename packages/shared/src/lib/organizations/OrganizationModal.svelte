<script lang="ts">
	import { getContext } from '$lib/context';
	import Loading from '$lib/network/Loading.svelte';
	import { isFound } from '$lib/network/loadable';
	import { OrganizationService } from '$lib/organizations/organizationService';
	import { getOrganizationBySlug } from '$lib/organizations/organizationsPreview.svelte';
	import { getProjectByRepositoryId } from '$lib/organizations/projectsPreview.svelte';
	import { AppState } from '$lib/redux/store.svelte';
	import { UserService } from '$lib/users/userService';
	import { getUserByLogin } from '$lib/users/usersPreview.svelte';
	import Button from '@gitbutler/ui/Button.svelte';
	import Modal from '@gitbutler/ui/Modal.svelte';
	import SectionCard from '@gitbutler/ui/SectionCard.svelte';
	import Textbox from '@gitbutler/ui/Textbox.svelte';
	import Avatar from '@gitbutler/ui/avatar/Avatar.svelte';

	type Props = {
		slug: string;
	};

	const { slug }: Props = $props();

	const appState = getContext(AppState);
	const organizationService = getContext(OrganizationService);
	const userService = getContext(UserService);

	const organization = $derived(getOrganizationBySlug(appState, organizationService, slug));

	const title = $derived.by(() => {
		if (!isFound(organization.current)) return '';
		return organization.current.value.name || organization.current.value.slug;
	});

	function onModalClose() {}

	let modal = $state<Modal>();
</script>

<Modal bind:this={modal} onClose={onModalClose} {title}>
	<Loading loadable={organization.current}>
		{#snippet children(organization)}
			{#if organization.inviteCode}
				<div class="header-with-action">
					<p>Invite code:</p>
					<Textbox value={organization.inviteCode} readonly></Textbox>
				</div>
			{/if}

			{#if organization.memberLogins}
				<h5 class="text-15 text-bold">Users:</h5>

				<div>
					{#each organization.memberLogins as login, index}
						{@const user = getUserByLogin(appState, userService, login)}

						<SectionCard
							roundedBottom={index + 1 === organization.memberLogins.length}
							roundedTop={index === 0}
							orientation="row"
						>
							<Loading loadable={user.current}>
								{#snippet children(user)}
									<Avatar
										size="medium"
										tooltip={user?.name || 'Unknown'}
										srcUrl={user?.avatarUrl || ''}
									/>
									<p>{user?.name}</p>
								{/snippet}
							</Loading>
						</SectionCard>
					{/each}
				</div>
			{/if}

			{#if organization.projectRepositoryIds}
				<h5 class="text-15 text-bold">Projects:</h5>
				<div>
					{#each organization.projectRepositoryIds as repositoryId, index}
						{@const project = getProjectByRepositoryId(repositoryId)}

						<SectionCard
							roundedBottom={index + 1 === organization.projectRepositoryIds.length}
							roundedTop={index === 0}
							orientation="row"
						>
							<Loading loadable={project.current}>
								{#snippet children(project)}
									<p>{project.name}</p>
								{/snippet}
							</Loading>
						</SectionCard>
					{/each}
				</div>
			{/if}
		{/snippet}
	</Loading>
</Modal>

<Button onclick={() => modal?.show()}>View</Button>

<style lang="postcss">
	.header-with-action {
		display: flex;
		align-items: center;
		justify-content: space-between;

		margin-bottom: 8px;
	}
</style>
