<script>
	import Chart from 'chart.js/auto';
	import ChartDataLabels from 'chartjs-plugin-datalabels';
	Chart.defaults.color = '#fff';

	export let datasets = [];
	export let labels = [];
	export let horizontal = false;

	let canvas = null;
	let chart = null;

	async function setupChart(datasets, labels, horizontal, canvas) {
		if (!canvas || !labels?.length || !datasets?.length) return;

		const data = {labels, datasets};

		const options = {
			indexAxis: horizontal ? 'y' : 'x',
			responsive: true,
			maintainAspectRatio: false,
			animation: true,
			layout: {
				padding: {
					right: 0,
				},
			},
			elements: {
				bar: {
					borderWidth: 2,
				},
			},
			plugins: {
				legend: {
					display: false,
				},
				datalabels: {
					anchor: 'center',
					align: 'center',
				},
			},
		};

		if (!chart) {
			chart = new Chart(canvas, {
				type: 'bar',
				data,
				options,
				plugins: [ChartDataLabels],
			});
		} else {
			chart.data = data;
			chart.options = options;
			chart.update();
		}
	}

	$: setupChart(datasets, labels, horizontal, canvas);
</script>

{#if datasets?.length && labels?.length}
	<section class="chart">
		<canvas class="chartjs" bind:this={canvas} />
	</section>
{/if}

<style>
	section {
		position: relative;
		margin: 1rem auto 0 auto;
		width: 100%;
		height: 100%;
	}

	canvas {
		width: 100% !important;
	}
</style>
