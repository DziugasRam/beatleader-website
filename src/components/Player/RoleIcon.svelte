<script>
	import {BL_ASSETS_CDN} from '../../network/queues/beatleader/page-queue';
	import createBeatSaverService from '../../services/beatmaps';
	import {createEventDispatcher} from 'svelte';

	export let role;
	export let allRoles;
	export let index = 0;
	export let mapperId = null;
	export let profileAppearance;
	export let editModel = null;

	let roleIcon;
	let roleIconClass;
	let roleDescription;
	let roleLink;
	let show = false;
	let cls = null;

	async function verifyMapper(mapperId) {
		if (!mapperId) return;

		let beatSaverService = createBeatSaverService();
		const mapperInfoValue = await beatSaverService.getMapper(mapperId);
		if (mapperInfoValue.verifiedMapper) {
			roleIcon = null;
			cls = null;
			roleIconClass = 'fas fa-hat-wizard';
			roleDescription = 'Verified maps creator';
			roleLink = 'https://beatsaver.com/profile/' + mapperId;
		}
	}

	function updateRoleIcon(role, mapperId, profileAppearance, edit) {
		cls = null;
		roleIcon = null;
		roleIconClass = null;
		show = false;

		switch (role) {
			case 'mapper':
				show = !profileAppearance || profileAppearance.includes('mapper');
				verifyMapper(mapperId);
				break;
			case 'rankedteam':
				show = !profileAppearance || profileAppearance.includes('rankedteam');
				roleIconClass = 'fas fa-balance-scale-right';
				roleDescription = 'Ranking team member';
				break;
			case 'qualityteam':
				show = !profileAppearance || profileAppearance.includes('qualityteam');
				roleIconClass = 'fas fa-microscope';
				roleDescription = 'Nomination Quality Team member';
				break;
			case 'juniorrankedteam':
				show = !profileAppearance || profileAppearance.includes('juniorrankedteam');
				roleIconClass = 'fas fa-balance-scale';
				roleDescription = 'Junior Ranking team member';
				break;
			case 'creator':
				show = !profileAppearance || profileAppearance.includes('creator');
				roleIconClass = 'fas fa-tools';
				roleDescription = 'BL creator';
				break;
			case 'admin':
				show = !profileAppearance || profileAppearance.includes('admin');
				roleIconClass = 'fas fa-user-shield';
				roleDescription = 'Administrator';
				break;
			case 'booster':
				show = profileAppearance && profileAppearance.includes('booster');
				roleIcon = BL_ASSETS_CDN + '/boostericon.png';
				roleDescription = 'Discord Server Booster';
				if (profileAppearance.includes('tipper') && allRoles.includes('tipper')) {
					roleIcon = BL_ASSETS_CDN + '/boosterpatreon1.png';
					roleDescription = 'Tier 1 Patreon supporter and Discord Server Booster ❤️';
				} else if (profileAppearance.includes('supporter') && allRoles.includes('supporter')) {
					roleIcon = BL_ASSETS_CDN + '/boosterpatreon2.png';
					roleDescription = 'Tier 2 Patreon supporter and Discord Server Booster ❤️';
				} else if (profileAppearance.includes('sponsor') && allRoles.includes('sponsor')) {
					roleIcon = BL_ASSETS_CDN + '/boosterpatreon3.png';
					roleDescription = 'Highest tier Patreon supporter. Crypto godge, Discord Server Booster and awesome person owerall 😎';
				}
				cls = 'player-role';
				break;
			case 'tipper':
				show = !profileAppearance || (profileAppearance.includes('tipper') && (!profileAppearance.includes('booster') || edit));
				roleIcon = BL_ASSETS_CDN + '/patreon1.png';
				roleDescription = 'Tier 1 Patreon supporter.';
				cls = 'player-role';
				break;
			case 'supporter':
				show = !profileAppearance || (profileAppearance.includes('supporter') && (!profileAppearance.includes('booster') || edit));
				roleIcon = BL_ASSETS_CDN + '/patreon2.png';
				roleDescription = 'Tier 2 Patreon supporter.';
				cls = 'player-role';
				break;
			case 'sponsor':
				show = !profileAppearance || (profileAppearance.includes('sponsor') && (!profileAppearance.includes('booster') || edit));
				roleIcon = BL_ASSETS_CDN + '/patreon3.png';
				roleDescription = 'Highest tier Patreon supporter. Crypto godge';
				cls = 'player-role';
				break;
			case 'warplane':
				show = !profileAppearance || profileAppearance.includes('warplane');
				roleIcon = null;
				roleIconClass = 'fas fa-plane';
				roleDescription = 'Warplane';
				break;
		}

		if (edit && cls && index > 0) {
			cls += ' edit';
		}
	}

	function onToggleRole(role, e) {
		if (!role?.length || !editModel) return;

		e.preventDefault();
		e.stopPropagation();

		if (!editModel.data.profileAppearance) editModel.data.profileAppearance = [];

		if (editModel.data.profileAppearance.includes(role)) {
			editModel.data.profileAppearance = editModel.data.profileAppearance.filter(s => s !== role);
			if (!editModel.data.profileAppearance.length) editModel.data.profileAppearance = null;
		} else editModel.data.profileAppearance = [...editModel.data.profileAppearance, role];
	}

	$: updateRoleIcon(role, mapperId, editModel?.data?.profileAppearance ?? profileAppearance, !!editModel);
</script>

{#if (show || !!editModel) && (roleIcon || roleIconClass)}
	{#if roleLink && roleIcon}
		<a class={cls} href={roleLink} class:disabled={!show} on:click={e => onToggleRole(role, e)}>
			<img class="role-icon" src={roleIcon} title={editModel ? 'Click to toggle' : roleDescription} alt="Role icon" />
		</a>
	{:else if roleLink && roleIconClass}
		<a class={cls} href={roleLink} class:disabled={!!editModel && !show} on:click={e => onToggleRole(role, e)}>
			<i class={roleIconClass} title={editModel ? 'Click to toggle' : roleDescription} />
		</a>
	{:else if roleIcon}
		<div class={cls} class:disabled={!!editModel && !show} on:click={e => onToggleRole(role, e)}>
			<img class="role-icon" src={roleIcon} title={editModel ? 'Click to toggle' : roleDescription} alt="Role icon" />
		</div>
	{:else if roleIconClass}
		<div class={cls} class:disabled={!!editModel && !show} on:click={e => onToggleRole(role, e)}>
			<i class={roleIconClass} title={editModel ? 'Click to toggle' : roleDescription} />
		</div>
	{/if}
{/if}

<style>
	.player-role {
		position: absolute;
		top: -2.5em;
		left: 2em;
		width: 6em;
		display: block;
	}

	.player-role.edit {
		top: 2em;
		height: 6em;
	}

	.role-icon {
		z-index: 5;
		width: 100%;
	}

	i {
		color: var(--beatleader-primary);
		font-size: 1.5em;
	}

	img,
	i {
		transition: all 200ms;
	}

	:global(.edit-enabled) img,
	:global(.edit-enabled) i {
		cursor: cell;
	}

	.disabled img,
	.disabled i {
		filter: grayscale(1);
		opacity: 0.25;
	}

	.disabled i {
		opacity: 0.75;
	}

	.disabled img:hover,
	.disabled i:hover {
		filter: none;
		opacity: 0.5;
	}
</style>
