<script>
	import {createEventDispatcher} from 'svelte';
	import createScoresStore from '../../stores/http/http-scores-store.js';
	import createAccountStore from '../../stores/beatleader/account';
	import createPlaylistStore from '../../stores/playlists';
	import createFailedScoresStore from '../../stores/beatleader/failed-scores';
	import {navigate} from 'svelte-routing';
	import {opt} from '../../utils/js';
	import {getContext} from 'svelte';
	import {scrollToTargetAdjusted} from '../../utils/browser';
	import SongScore from './SongScore.svelte';
	import FailedScore from './FailedScore.svelte';
	import Error from '../Common/Error.svelte';
	import ScoreServiceSwitcher from './ScoreServiceSwitcher.svelte';
	import ScoresPager from './ScoresPager.svelte';
	import stringify from 'json-stable-stringify';
	import Pager from '../Common/Pager.svelte';
	import Button from '../Common/Button.svelte';
	import Spinner from '../Common/Spinner.svelte';
	import RangeSlider from 'svelte-range-slider-pips';
	import OpDeletionDialog from './OPDeletionDialog.svelte';
	import {BL_API_URL} from '../../network/queues/beatleader/api-queue.js';

	const dispatch = createEventDispatcher();
	const {open, close} = getContext('simple-modal');

	export let playerId = null;
	export let player = null;
	export let initialState = null;
	export let initialStateType = null;
	export let initialService = 'beatleader';
	export let initialServiceParams = {};
	export let numOfScores = null;
	export let fixedBrowserTitle = null;
	export let withPlayers = false;
	export let noIcons = false;

	let scoresStore = createScoresStore(playerId, initialService, initialServiceParams, initialState, initialStateType);

	const account = createAccountStore();
	const playlists = createPlaylistStore();

	let scoresBoxEl = null;

	function changeParams(newPlayerId, newService, newServiceParams) {
		if (!newPlayerId) return null;

		scoresStore.fetch(newServiceParams, newService, newPlayerId);

		return {playerId: newPlayerId, service: newService, serviceParams: newServiceParams};
	}

	let currentPage = 0;
	let previousPage = 0;

	function onPageChanged(event) {
		if (!(event?.detail?.initial ?? false)) scrollToTop();

		const page = (event?.detail?.page ?? 0) + 1;

		previousPage = currentPage;
		currentPage = page;

		if (!(event?.detail?.initial ?? false)) {
			dispatch('page-changed', page);
		}
	}

	function onServiceParamsChanged(event) {
		if (!event?.detail) return;

		dispatch('service-params-changed', event.detail);
	}

	function onServiceChanged(event) {
		if (!event?.detail) return;

		scrollToTop();

		dispatch('service-changed', event.detail);
	}

	function scrollToTop() {
		if (scoresBoxEl) scrollToTargetAdjusted(scoresBoxEl, 44);
	}

	let currentService = null;
	let lastService = '';
	function updateService(scoresStore) {
		if (!scoresStore) return;

		const newService = scoresStore.getService();
		if (lastService !== newService) currentService = newService;

		lastService = newService;
	}

	let currentServiceParams = null;
	let lastServiceParams = '';
	function updateServiceParams(scoresStore) {
		if (!scoresStore) return;

		const newServiceParams = stringify(scoresStore.getServiceParams());
		if (lastServiceParams !== newServiceParams) currentServiceParams = scoresStore.getServiceParams();

		lastServiceParams = newServiceParams;
	}

	function onFailedScoresPageChange(event) {
		const page = (event?.detail?.page ?? 0) + 1;

		failedScores.fetchScores(page);
	}

	const failedScores = createFailedScoresStore();

	let searchToPlaylist = false;
	let makingPlaylist = false;
	let mapCount = 100;
	let duplicateDiffs = false;
	function generatePlaylist() {
		makingPlaylist = true;
		playlists.generatePlayerPlaylist(mapCount, playerId, {...currentServiceParams, duplicateDiffs}, () => {
			navigate('/playlists');
		});
	}

	$: changeParams(playerId, initialService, initialServiceParams, initialState, initialStateType);
	$: $scoresStore, updateService(scoresStore);
	$: $scoresStore, updateServiceParams(scoresStore);
	$: page = currentServiceParams?.page ?? null;
	$: totalScores = (scoresStore => (scoresStore && scoresStore.getTotalScores ? scoresStore.getTotalScores() : null))(
		scoresStore,
		$scoresStore
	);
	$: pending = scoresStore ? scoresStore.pending : null;
	$: error = scoresStore ? scoresStore.error : null;
	$: isMain = playerId && $account?.id === playerId;
	$: isMain ? failedScores.refresh() : null;
	$: isAdmin = $account?.player?.playerInfo?.role?.includes('admin');

	$: failedScoresPage = opt($failedScores, 'metadata.page');
	$: totalFailedScores = opt($failedScores, 'metadata.total');
	$: failedScoresArray = opt($failedScores, 'scores');

	$: scoresStore && scoresStore.fetch(currentServiceParams, currentService);
	$: pagerTotalScores = totalScores !== null && totalScores !== undefined ? totalScores : numOfScores;

	$: itemsPerPage = (itemsPerPage => (itemsPerPage && itemsPerPage.getItemsPerPage ? scoresStore.getItemsPerPage() : null))(
		scoresStore,
		$scoresStore
	);

	let waiting = false;

	function removeOPScores() {
		waiting = true;
		fetch(BL_API_URL + 'user/hideopscores', {
			credentials: 'include',
			method: 'POST',
		}).then(() => {
			waiting = false;
			document.location.reload();
		});
	}

	function showRemoveOP() {
		open(OpDeletionDialog, {
			confirm: () => {
				close();
				removeOPScores();
			},
			cancel: () => {
				close();
			},
		});
	}

	$: OPScores = isMain && $scoresStore?.length && $scoresStore.find(s => s.score?.mods?.includes('OP'));
</script>

<div bind:this={scoresBoxEl}>
	{#if $error}
		<div><Error error={$error} /></div>
	{/if}

	<ScoreServiceSwitcher
		{playerId}
		{player}
		service={currentService}
		serviceParams={currentServiceParams}
		loadingService={$pending?.service}
		loadingServiceParams={$pending?.serviceParams}
		on:service-change={onServiceChanged}
		on:service-params-change={onServiceParamsChanged} />

	{#if $scoresStore && $scoresStore.length}
		<div class="song-scores grid-transition-helper">
			{#each $scoresStore as songScore, idx ((songScore?.id ?? '') + (songScore?.score?.id ?? ''))}
				<SongScore
					{playerId}
					{songScore}
					{fixedBrowserTitle}
					{idx}
					service={currentService}
					{withPlayers}
					{noIcons}
					animationSign={currentPage >= previousPage ? 1 : -1}
					additionalStat={currentServiceParams?.sort} />
			{/each}
		</div>
	{:else}
		<p>No scores.</p>
	{/if}

	{#if currentService == 'beatleader'}
		<Button
			cls={pagerTotalScores > itemsPerPage ? 'scores-playlist-button' : 'scores-playlist-button-relative'}
			iconFa="fas fa-list"
			type={searchToPlaylist ? 'danger' : 'default'}
			label={searchToPlaylist ? 'Cancel' : 'To Playlist!'}
			on:click={() => (searchToPlaylist = !searchToPlaylist)} />
		{#if searchToPlaylist}
			{#if makingPlaylist}
				<Spinner />
			{:else}
				<span>Maps count:</span>
				<RangeSlider
					range
					min={0}
					max={1000}
					step={1}
					values={[mapCount]}
					hoverable
					float
					pips
					pipstep={100}
					all="label"
					on:change={event => {
						mapCount = event.detail.values[0];
					}} />
				<div class="duplicateDiffsContainer">
					<input type="checkbox" id="duplicateDiffs" label="Duplicate map per diff" bind:checked={duplicateDiffs} />
					<label for="duplicateDiffs" title="Will include every diff as a separate map entry">Duplicate map per diff</label>
				</div>
				<Button cls="playlist-button" iconFa="fas fa-wand-magic-sparkles" label="Generate playlist" on:click={() => generatePlaylist()} />
			{/if}
		{/if}
	{/if}

	{#if Number.isFinite(page) && (!Number.isFinite(pagerTotalScores) || pagerTotalScores > 0)}
		<ScoresPager
			{playerId}
			service={currentService}
			serviceParams={currentServiceParams}
			totalItems={pagerTotalScores}
			currentPage={page - 1}
			fixedItemsPerPage={itemsPerPage}
			loadingPage={$pending?.serviceParams?.page ? $pending.serviceParams.page - 1 : null}
			on:page-changed={onPageChanged} />
	{/if}

	{#if OPScores}
		{#if waiting}
			<Spinner />
		{:else}
			<Button label="Remove OP scores" type="danger" iconFa="fas fa-trash-alt" on:click={showRemoveOP} />
		{/if}
	{/if}

	{#if isMain && failedScoresArray && failedScoresArray.length}
		<div class="song-scores failed-scores grid-transition-helper">
			{#each failedScoresArray as songScore, idx (opt(songScore, 'score.id'))}
				<FailedScore store={failedScores} {playerId} {songScore} {fixedBrowserTitle} {idx} service={currentService} {isAdmin} />
			{/each}
		</div>
		{#if Number.isFinite(failedScoresPage) && (!Number.isFinite(totalFailedScores) || totalFailedScores > 0)}
			<Pager
				totalItems={totalFailedScores}
				itemsPerPage={3}
				itemsPerPageValues={null}
				currentPage={failedScoresPage - 1}
				on:page-changed={onFailedScoresPageChange} />
		{/if}
	{/if}
</div>

<style>
	.song-scores :global(> *:last-child) {
		border-bottom: none !important;
	}

	:global(.scores-playlist-button) {
		height: 1.6em;
		position: absolute !important;
		right: 1em;
		margin-top: 0.6em !important;
	}

	:global(.scores-playlist-button-relative) {
		height: 1.6em;
		margin-top: 0.6em !important;
	}

	.duplicateDiffsContainer {
		display: flex;
	}

	#duplicateDiffs {
		width: auto;
	}
	@media screen and (max-width: 768px) {
		:global(.scores-playlist-button) {
			margin-top: 9em !important;
			right: auto;
		}
	}
</style>
