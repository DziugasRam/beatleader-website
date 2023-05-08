<script>
	import {fly} from 'svelte/transition';
	import {inview} from 'svelte-inview';
	import Scroller from '@sveltejs/svelte-scroller';
	import ssrConfig from '../ssr-config';
	import ContentBox from '../components/Common/ContentBox.svelte';
	import BarChart from '../components/Census/BarChart.svelte';

	document.body.classList.add('slim');

	const inViewOptions = {rootMargin: '-50px'};

	let demographicsInView = false;
	let demographicsIndex = null;
	let prevDemographicsIndex = null;
	const demographicsData = [
		{
			labels: ['a', 'b', 'c', 'd', 'e', 'f'],
			datasets: [
				{
					data: [100, 200, 300, 400, 300, 200],
					borderWidth: 0,
					backgroundColor: 'red',
					borderRadius: 16,
				},
			],
			horizontal: true,
		},
		{
			labels: ['a', 'b', 'c', 'd', 'e', 'f'],
			datasets: [
				{
					data: [300, null, null, 50, 200, 300],
					borderWidth: 0,
					backgroundColor: 'green',
					grouped: true,
					stack: 0,
					borderRadius: 16,
				},
				{
					data: [null, -150, -100, null, null, null],
					borderWidth: 0,
					backgroundColor: 'red',
					grouped: true,
					stack: 0,
					borderRadius: 16,
				},
			],
			horizontal: true,
		},
		{
			labels: ['a', 'b', 'c', 'd', 'e', 'f'],
			datasets: [
				{
					data: [100, 200, 300, 400, 300, 200],
					borderWidth: 0,
					backgroundColor: 'blue',
				},
			],
		},
	];

	$: if (demographicsIndex !== prevDemographicsIndex) {
		prevDemographicsIndex = demographicsIndex;
	}

	let backgroundInView = false;
	let backgroundIndex = null;
	let prevbackgroundIndex = null;
	const backgroundData = [
		{
			labels: ['a', 'b', 'c', 'd', 'e', 'f'],
			datasets: [
				{
					data: [100, 200, 300, 400, 300, 200],
					borderWidth: 0,
					backgroundColor: 'salmon',
					borderRadius: 16,
				},
			],
			horizontal: true,
		},
		{
			labels: ['a', 'b', 'c', 'd', 'e', 'f'],
			datasets: [
				{
					data: [300, null, null, 50, 200, 300],
					borderWidth: 0,
					backgroundColor: 'green',
					grouped: true,
					stack: 0,
					borderRadius: 16,
				},
				{
					data: [null, -150, -100, null, null, null],
					borderWidth: 0,
					backgroundColor: 'red',
					grouped: true,
					stack: 0,
					borderRadius: 16,
				},
			],
			horizontal: true,
		},
		{
			labels: ['a', 'b', 'c', 'd', 'e', 'f'],
			datasets: [
				{
					data: [100, 200, 300, 400, 300, 200],
					borderWidth: 0,
					backgroundColor: 'orange',
				},
			],
		},
	];

	$: if (backgroundIndex !== prevbackgroundIndex) {
		prevbackgroundIndex = backgroundIndex;
	}
</script>

<svelte:head>
	<title>Beat Saber in numbers - {ssrConfig.name}</title>
</svelte:head>

<article>
	<ContentBox>
		<h1 class="title is-3">Beat Saber in numbers</h1>

		<p>
			Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
			minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in
			reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
			culpa qui officia deserunt mollit anim id est laborum.
		</p>

		<p>
			Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
			minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in
			reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
			culpa qui officia deserunt mollit anim id est laborum.
		</p>

		<h2 class="title is-4">Demographics data</h2>

		<div class="census-chart-container" use:inview={inViewOptions} on:inview_enter={e => (demographicsInView = true)}>
			{#if demographicsInView}
				<Scroller top={0} bottom={1} threshold={0.65} bind:index={demographicsIndex}>
					<div slot="background">
						<div class="census-chart" transition:fly={{duration: 500, delay: 500, x: 300}}>
							<BarChart {...demographicsData?.[demographicsIndex]} />
						</div>
					</div>

					<div slot="foreground">
						<section>
							<span>This is the first note about dataset</span>
						</section>

						<section>
							<span>This is the second note about dataset</span>
						</section>

						<section>
							<span>This is the third note about dataset</span>
						</section>
					</div>
				</Scroller>
			{/if}
		</div>

		<p>
			Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
			minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in
			reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
			culpa qui officia deserunt mollit anim id est laborum.
		</p>

		<p>
			Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
			minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in
			reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
			culpa qui officia deserunt mollit anim id est laborum.
		</p>

		<h2 class="title is-4">Background data</h2>

		<div class="census-chart-container" use:inview={inViewOptions} on:inview_enter={e => (backgroundInView = true)}>
			{#if backgroundInView}
				<Scroller top={0} bottom={1} threshold={0.65} bind:index={backgroundIndex}>
					<div slot="background">
						<div class="census-chart" transition:fly={{duration: 500, delay: 500, x: 300}}>
							<BarChart {...backgroundData?.[backgroundIndex]} />
						</div>
					</div>

					<div slot="foreground">
						<section>
							<span>This is the first note about dataset</span>
						</section>

						<section>
							<span>This is the second note about dataset</span>
						</section>

						<section>
							<span>This is the third note about dataset</span>
						</section>
					</div>
				</Scroller>
			{/if}
		</div>

		<p>
			Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
			minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in
			reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
			culpa qui officia deserunt mollit anim id est laborum.
		</p>

		<p>
			Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad
			minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in
			reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in
			culpa qui officia deserunt mollit anim id est laborum.
		</p>
	</ContentBox>
</article>

<style>
	article {
		max-width: 65rem;
		margin-inline: auto;
	}

	article :global(.content-box) {
		backdrop-filter: none !important;
	}

	[slot='background'] {
		overflow: hidden;
	}

	[slot='foreground'] {
		pointer-events: none;
	}

	[slot='foreground'] section {
		padding-top: 3rem;
		height: 100vh;
		pointer-events: all;
	}

	[slot='foreground'] section > span {
		display: inline-block;
		background-color: rgba(0, 0, 0, 0.5);
		color: var(--textColor);
		padding: 1rem;
	}

	p {
		margin-bottom: 1rem;
	}

	.census-chart-container {
		min-height: 100dvh;
	}

	.census-chart {
		min-height: calc(100dvh - 1rem - 3rem);
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>
