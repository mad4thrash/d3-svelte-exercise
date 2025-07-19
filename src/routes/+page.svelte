<script lang="ts">
	type Margin = {
		top: number;
		bottom: number;
		left: number;
		right: number;
	};

	let data = $state<[]>([]);

	const margin: Margin = {
		top: 50,
		bottom: 50,
		left: 50,
		right: 50
	};

	let chartWidth: number = $state<number>(400);
	let chartHeight: number = 400;
	let innerChartWidth: number = $derived(chartWidth - margin.left - margin.right);
	let innerChartHeight: number = $derived(chartHeight - margin.top - margin.bottom);

	$effect(() => {
		console.log('ciao');
	});
</script>

<div class="flex h-svh w-full items-center justify-center" bind:clientWidth={chartWidth}>
	<svg class="bg-white" width={chartWidth} height={chartHeight}>
		<g style="transform: translate({margin.left}px, {margin.top}px)">
			<line
				x1="0"
				y1={innerChartHeight}
				x2={innerChartWidth}
				y2={innerChartHeight}
				stroke="black"
				stroke-width="4px"
			/>
			<line x1="0" y1="0" x2="0" y2={innerChartHeight} stroke="black" stroke-width="4px" />
		</g>
	</svg>
</div>
