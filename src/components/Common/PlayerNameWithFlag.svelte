<script>
	import {createEventDispatcher} from 'svelte';
	import {opt} from '../../utils/js';
	import Flag from './Flag.svelte';
	import Popover from './Popover.svelte';
	import {fade} from 'svelte/transition';
	import MiniProfile from '../Player/MiniProfile.svelte';

	export let player;
	export let type = null;
	export let hideFlag = false;
	export let withCrown = false;
	export let playerClickFilter = null;
	export let disablePopover = false;

	const dispatch = createEventDispatcher();

	let referenceElement;

	$: country = opt(player, 'playerInfo.countries.0.country') ?? player?.country;
	$: name = player?.name;
	$: playerId = player?.playerId ?? player?.id;
</script>

<a
	href={`/u/${playerId}${type ? '/' + type : ''}/1?${playerClickFilter ?? ''}`}
	class="player-name clickable has-pointer-events"
	title={name}
	bind:this={referenceElement}
	on:click|preventDefault>
	{#if !hideFlag}
		<Flag {country} on:flag-click />
	{/if}
	<span class="name"
		>{#if withCrown}<span class="crown">👑</span>{/if}{name ?? 'Unknown'}</span>
</a>

{#if !disablePopover && player && player.playerInfo}
	<Popover triggerEvents={['hover', 'focus']} {referenceElement} placement="top" spaceAway={10}>
		<div class="popover-contents" transition:fade={{duration: 250}}>
			<MiniProfile {player} />
		</div>
	</Popover>
{/if}

<style>
	a {
		color: inherit !important;
	}

	.player-name {
		white-space: nowrap;
		overflow-x: hidden;
		overflow: hidden;
		word-break: break-all;
	}

	.player-name :global(> img) {
		margin-right: 0.125rem;
	}

	.crown {
		position: relative;
		top: -0.125em;
	}

	.popover-contents {
		width: 40em;
	}
</style>
