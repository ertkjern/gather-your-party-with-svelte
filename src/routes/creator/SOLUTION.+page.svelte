<script lang="ts">
	import CharacterCreatorDebug from '$lib/_workshop-internals/components/character-creator-debug/CharacterCreatorDebug.svelte';
	import AttributeOverview from '$lib/components/attributes/attribute-overview/AttributeOverview.svelte';
	import Stats from '$lib/components/attributes/stats/Stats.svelte';
	import Button from '$lib/components/button/Button.svelte';
	import PortraitPicker from '$lib/components/portraits/portrait-picker/PortraitPicker.svelte';
	import RaceSelect from '$lib/components/race-select/RaceSelect.svelte';
	import SkillSelect from '$lib/components/skill-select/SkillSelect.svelte';
	import { INITIAL_ATTRIBUTES } from '$lib/constants/initial-attributes';
	import { CLASSES } from '$lib/models/classes';
	import type { AttributeAllocation } from '$lib/types/attribute-allocation';
	import type { Character } from '$lib/types/character';
	import type { Class } from '$lib/types/class';
	import type { Race } from '$lib/types/race';
	import type { Skill } from '$lib/types/skill';
	import type { UpsertCharacterRequest } from '$lib/types/upsert-character-request';
	import { apiFetch } from '$lib/utils/api-fetch';

	let saveCharacterPromise: Promise<Character>;
	let isSavingCharacter = false;

	let name = '';

	let race: Race;

	type ClassOption = { id: Class; name: string };
	const CLASS_OPTIONS: ClassOption[] = Object.entries(CLASSES).map(([name, id]) => ({ id, name }));

	let primaryClass: Class;
	let isDualClass = false;
	let secondaryClass: Class;

	const handleSelectPrimaryClass = () => {
		secondaryClass = null;
	};

	let portrait: string;

	let attributes: AttributeAllocation = INITIAL_ATTRIBUTES;

	let skills: Skill[];

	$: character = {
		name,
		race,
		primaryClass,
		secondaryClass,
		portrait,
		attributes,
		skills,
	} as UpsertCharacterRequest;

	const handleSubmit = () => {
		isSavingCharacter = true;

		saveCharacterPromise = apiFetch<Character>('/api/characters', {
			method: 'POST',
			body: JSON.stringify(character),
		}).finally(() => {
			isSavingCharacter = false;
		});
	};
</script>

<svelte:head>
	<title>Character Creator</title>
</svelte:head>

<CharacterCreatorDebug {character} />

<section class="rpgui-container framed outer-container">
	<div class="container">
		<label class="input">
			<span>Name</span>
			<input class="input-content" bind:value={name} />
		</label>

		<label class="input" for="race">
			<span data-quest-1="label">Race</span>
			<RaceSelect id="race" bind:value={race} />
		</label>

		<label class="input">
			<span>Class</span>
			<select
				class="rpgui-dropdown-imp input-content"
				bind:value={primaryClass}
				on:change={handleSelectPrimaryClass}
			>
				<option value={null}>Please select...</option>

				{#each CLASS_OPTIONS as { id, name }}
					<option value={id}>{name}</option>
				{/each}
			</select>

			<div class="suffix">
				<input
					id="is-dual-class"
					class="rpgui-checkbox"
					type="checkbox"
					bind:checked={isDualClass}
				/>
				<label for="is-dual-class">Dual class?</label>
			</div>
		</label>

		{#if isDualClass}
			<label class="input">
				<span>Dual class</span>
				<select class="rpgui-dropdown-imp input-content" bind:value={secondaryClass}>
					<option value={null}>Please select...</option>

					{#each CLASS_OPTIONS as { id, name }}
						<option value={id} disabled={id === primaryClass}>{name}</option>
					{/each}
				</select>
			</label>
		{/if}

		<label class="input" for="portrait">
			<span data-quest-6="label">Portrait</span>
			<PortraitPicker id="portrait" bind:value={portrait} />
		</label>

		<label class="input" for="attributes">
			<span data-quest-5>Attributes</span>
			<AttributeOverview id="attributes" bind:value={attributes} />
		</label>

		<label class="input" for="stats">
			<span data-quest-3>Stats</span>
			<Stats id="stats" {attributes} />
		</label>

		<label class="input" for="skills">
			<span data-quest-4>Skills</span>
			<SkillSelect id="skills" bind:value={skills} />
		</label>
	</div>

	<Button variant="golden" disabled={isSavingCharacter} on:click={handleSubmit}>
		{isSavingCharacter ? 'Saving character...' : 'Save character'}
	</Button>

	{#await saveCharacterPromise then savedCharacter}
		{#if savedCharacter}
			&check; Character "{savedCharacter.name}" saved.
		{/if}
	{:catch _}
		&cross; Error saving character, see console for more info.
	{/await}
</section>

<style>
	.outer-container {
		display: flex;
		flex-direction: column;
		align-items: center;
		gap: 1rem;
	}

	.container {
		display: flex;
		flex-direction: column;
		gap: 2rem;
		width: calc(100% - 2rem);
	}

	.input {
		position: relative;
		display: flex;
		justify-content: stretch;
	}

	.input > span {
		width: 12rem;
	}

	.input-content {
		margin: auto;
	}

	.suffix {
		position: absolute;
		right: -1rem;
	}
</style>
