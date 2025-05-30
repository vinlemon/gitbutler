<script lang="ts">
	import AIPromptEdit from '$components/AIPromptEdit.svelte';
	import AuthorizationBanner from '$components/AuthorizationBanner.svelte';
	import InfoMessage from '$components/InfoMessage.svelte';
	import Section from '$components/Section.svelte';
	import { AISecretHandle, AIService, GitAIConfigKey, KeyOption } from '$lib/ai/service';
	import { OpenAIModelName, AnthropicModelName, ModelKind } from '$lib/ai/types';
	import { GitConfigService } from '$lib/config/gitConfigService';
	import { getSecretsService } from '$lib/secrets/secretsService';
	import { UserService } from '$lib/user/userService';
	import { getContext } from '@gitbutler/shared/context';
	import RadioButton from '@gitbutler/ui/RadioButton.svelte';
	import SectionCard from '@gitbutler/ui/SectionCard.svelte';
	import Spacer from '@gitbutler/ui/Spacer.svelte';
	import Textbox from '@gitbutler/ui/Textbox.svelte';
	import Link from '@gitbutler/ui/link/Link.svelte';
	import Select from '@gitbutler/ui/select/Select.svelte';
	import SelectItem from '@gitbutler/ui/select/SelectItem.svelte';
	import { onMount, tick } from 'svelte';
	import { run } from 'svelte/legacy';

	const gitConfigService = getContext(GitConfigService);
	const secretsService = getSecretsService();
	const aiService = getContext(AIService);
	const userService = getContext(UserService);
	const user = userService.user;
	let initialized = false;

	let modelKind: ModelKind | undefined = $state();
	let openAIKeyOption: KeyOption | undefined = $state();
	let anthropicKeyOption: KeyOption | undefined = $state();
	let openAIKey: string | undefined = $state();
	let openAIModelName: OpenAIModelName | undefined = $state();
	let anthropicKey: string | undefined = $state();
	let anthropicModelName: AnthropicModelName | undefined = $state();
	let diffLengthLimit: number | undefined = $state();
	let ollamaEndpoint: string | undefined = $state();
	let ollamaModel: string | undefined = $state();

	async function setConfiguration(key: GitAIConfigKey, value: string | undefined) {
		if (!initialized) return;
		gitConfigService.set(key, value || '');
	}

	async function setSecret(handle: AISecretHandle, secret: string | undefined) {
		if (!initialized) return;
		await secretsService.set(handle, secret || '');
	}

	onMount(async () => {
		modelKind = await aiService.getModelKind();

		openAIKeyOption = await aiService.getOpenAIKeyOption();
		openAIModelName = await aiService.getOpenAIModleName();
		openAIKey = await aiService.getOpenAIKey();

		anthropicKeyOption = await aiService.getAnthropicKeyOption();
		anthropicModelName = await aiService.getAnthropicModelName();
		anthropicKey = await aiService.getAnthropicKey();

		diffLengthLimit = await aiService.getDiffLengthLimit();

		ollamaEndpoint = await aiService.getOllamaEndpoint();
		ollamaModel = await aiService.getOllamaModelName();

		// Ensure reactive declarations have finished running before we set initialized to true
		await tick();

		initialized = true;
	});

	const keyOptions = [
		{
			label: 'Use GitButler API',
			value: KeyOption.ButlerAPI
		},
		{
			label: 'Your own key',
			value: KeyOption.BringYourOwn
		}
	];

	const openAIModelOptions = [
		{
			label: 'o3 Mini',
			value: OpenAIModelName.O3mini
		},
		{
			label: 'o1 Mini',
			value: OpenAIModelName.O1mini
		},
		{
			label: 'GPT 4o mini (recommended)',
			value: OpenAIModelName.GPT4oMini
		}
	];

	const anthropicModelOptions = [
		{
			label: 'Haiku',
			value: AnthropicModelName.Haiku
		},
		{
			label: 'Sonnet 3.5',
			value: AnthropicModelName.Sonnet35
		},
		{
			label: 'Sonnet 3.7 (recommended)',
			value: AnthropicModelName.Sonnet37
		}
	];

	let form = $state<HTMLFormElement>();

	function onFormChange(form: HTMLFormElement) {
		const formData = new FormData(form);
		modelKind = formData.get('modelKind') as ModelKind;
	}
	run(() => {
		setConfiguration(GitAIConfigKey.ModelProvider, modelKind);
	});
	run(() => {
		setConfiguration(GitAIConfigKey.OpenAIKeyOption, openAIKeyOption);
	});
	run(() => {
		setConfiguration(GitAIConfigKey.OpenAIModelName, openAIModelName);
	});
	run(() => {
		setSecret(AISecretHandle.OpenAIKey, openAIKey);
	});
	run(() => {
		setConfiguration(GitAIConfigKey.AnthropicKeyOption, anthropicKeyOption);
	});
	run(() => {
		setConfiguration(GitAIConfigKey.AnthropicModelName, anthropicModelName);
	});
	run(() => {
		setConfiguration(GitAIConfigKey.DiffLengthLimit, diffLengthLimit?.toString());
	});
	run(() => {
		setSecret(AISecretHandle.AnthropicKey, anthropicKey);
	});
	run(() => {
		setConfiguration(GitAIConfigKey.OllamaEndpoint, ollamaEndpoint);
	});
	run(() => {
		setConfiguration(GitAIConfigKey.OllamaModelName, ollamaModel);
	});
	run(() => {
		if (form) form.modelKind.value = modelKind;
	});
</script>

<p class="text-13 text-body ai-settings__text">
	GitButler supports multiple providers for its AI powered features. We currently support models
	from OpenAI and Anthropic either proxied through the GitButler API, or in a bring your own key
	configuration.
</p>

<form class="git-radio" bind:this={form} onchange={(e) => onFormChange(e.currentTarget)}>
	<SectionCard
		roundedBottom={false}
		orientation="row"
		labelFor="open-ai"
		bottomBorder={modelKind !== ModelKind.OpenAI}
	>
		{#snippet title()}
			Open AI
		{/snippet}
		{#snippet actions()}
			<RadioButton name="modelKind" id="open-ai" value={ModelKind.OpenAI} />
		{/snippet}
	</SectionCard>
	{#if modelKind === ModelKind.OpenAI}
		<SectionCard roundedTop={false} roundedBottom={false} topDivider>
			<Select
				value={openAIKeyOption}
				options={keyOptions}
				wide
				label="Do you want to provide your own key?"
				onselect={(value) => {
					openAIKeyOption = value as KeyOption;
				}}
			>
				{#snippet itemSnippet({ item, highlighted })}
					<SelectItem selected={item.value === openAIKeyOption} {highlighted}>
						{item.label}
					</SelectItem>
				{/snippet}
			</Select>

			{#if openAIKeyOption === KeyOption.ButlerAPI}
				{#if !$user}
					<AuthorizationBanner message="Please sign in to use the GitButler API." />
				{:else}
					<InfoMessage filled outlined={false} style="pop" icon="ai">
						{#snippet title()}
							GitButler uses OpenAI API for commit messages and branch names
						{/snippet}
					</InfoMessage>
				{/if}
			{/if}

			{#if openAIKeyOption === KeyOption.BringYourOwn}
				<Textbox label="API key" bind:value={openAIKey} required placeholder="sk-..." />

				<Select
					value={openAIModelName}
					options={openAIModelOptions}
					label="Model version"
					wide
					onselect={(value) => {
						openAIModelName = value as OpenAIModelName;
					}}
				>
					{#snippet itemSnippet({ item, highlighted })}
						<SelectItem selected={item.value === openAIModelName} {highlighted}>
							{item.label}
						</SelectItem>
					{/snippet}
				</Select>
			{/if}
		</SectionCard>
	{/if}

	<SectionCard
		roundedTop={false}
		roundedBottom={false}
		orientation="row"
		labelFor="anthropic"
		bottomBorder={modelKind !== ModelKind.Anthropic}
	>
		{#snippet title()}
			Anthropic
		{/snippet}
		{#snippet actions()}
			<RadioButton name="modelKind" id="anthropic" value={ModelKind.Anthropic} />
		{/snippet}
	</SectionCard>
	{#if modelKind === ModelKind.Anthropic}
		<SectionCard roundedTop={false} roundedBottom={false} topDivider>
			<Select
				value={anthropicKeyOption}
				options={keyOptions}
				wide
				label="Do you want to provide your own key?"
				onselect={(value) => {
					anthropicKeyOption = value as KeyOption;
				}}
			>
				{#snippet itemSnippet({ item, highlighted })}
					<SelectItem selected={item.value === anthropicKeyOption} {highlighted}>
						{item.label}
					</SelectItem>
				{/snippet}
			</Select>

			{#if anthropicKeyOption === KeyOption.ButlerAPI}
				{#if !$user}
					<AuthorizationBanner message="Please sign in to use the GitButler API." />
				{:else}
					<InfoMessage filled outlined={false} style="pop" icon="ai">
						{#snippet title()}
							GitButler uses Anthropic API for commit messages and branch names
						{/snippet}
					</InfoMessage>
				{/if}
			{/if}

			{#if anthropicKeyOption === KeyOption.BringYourOwn}
				<Textbox
					label="API key"
					bind:value={anthropicKey}
					required
					placeholder="sk-ant-api03-..."
				/>

				<Select
					value={anthropicModelName}
					options={anthropicModelOptions}
					label="Model version"
					onselect={(value) => {
						anthropicModelName = value as AnthropicModelName;
					}}
				>
					{#snippet itemSnippet({ item, highlighted })}
						<SelectItem selected={item.value === anthropicModelName} {highlighted}>
							{item.label}
						</SelectItem>
					{/snippet}
				</Select>
			{/if}
		</SectionCard>
	{/if}

	<SectionCard
		roundedTop={false}
		roundedBottom={modelKind !== ModelKind.Ollama}
		orientation="row"
		labelFor="ollama"
		bottomBorder={modelKind !== ModelKind.Ollama}
	>
		{#snippet title()}
			Ollama 🦙
		{/snippet}
		{#snippet actions()}
			<RadioButton name="modelKind" id="ollama" value={ModelKind.Ollama} />
		{/snippet}
	</SectionCard>
	{#if modelKind === ModelKind.Ollama}
		<SectionCard roundedTop={false} topDivider>
			<Textbox label="Endpoint" bind:value={ollamaEndpoint} placeholder="http://127.0.0.1:11434" />

			<Textbox label="Model" bind:value={ollamaModel} placeholder="llama3" />
		</SectionCard>
		<SectionCard roundedTop={false} topDivider>
			<InfoMessage filled outlined={false}>
				{#snippet content()}
					<p class="text-13">
						Connecting to your Ollama endpoint will <b
							>require you to allow-list it in the CSP settings for the application.</b
						>
						<br />
						<br />
						You can find more details on how to do that in the <Link
							href="https://docs.gitbutler.com/troubleshooting/custom-csp">docs</Link
						>
					</p>
				{/snippet}
			</InfoMessage>
		</SectionCard>
	{/if}
</form>

<Spacer />

<SectionCard orientation="row">
	{#snippet title()}
		Amount of provided context
	{/snippet}
	{#snippet caption()}
		How many characters of your git diff should be provided to AI
	{/snippet}
	{#snippet actions()}
		<Textbox
			type="number"
			width={80}
			textAlign="center"
			value={diffLengthLimit?.toString()}
			minVal={100}
			oninput={(value: string) => {
				diffLengthLimit = parseInt(value);
			}}
			placeholder="5000"
		/>
	{/snippet}
</SectionCard>

<Spacer />

<Section>
	{#snippet title()}
		Custom AI prompts
	{/snippet}
	{#snippet description()}
		GitButler's AI assistant generates commit messages and branch names. Use default prompts or
		create your own. Assign prompts in the project settings.
	{/snippet}

	<div class="prompt-groups">
		<AIPromptEdit promptUse="commits" />
		<Spacer margin={12} />
		<AIPromptEdit promptUse="branches" />
	</div>
</Section>

<style>
	.ai-settings__text {
		margin-bottom: 12px;
		color: var(--clr-text-2);
	}

	.prompt-groups {
		display: flex;
		flex-direction: column;
		margin-top: 16px;
		gap: 12px;
	}
</style>
