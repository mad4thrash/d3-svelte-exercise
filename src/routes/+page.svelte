<script lang="ts">
	import * as d3 from 'd3';

	type Margin = {
		top: number;
		bottom: number;
		left: number;
		right: number;
	};

	let data = $state<any[]>([]);

	const margin: Margin = {
		top: 50,
		bottom: 70,
		left: 70,
		right: 50
	};

	let chartWidth: number = $state<number>(400);
	let chartHeight: number = 400;
	let innerChartWidth: number = $derived(chartWidth - margin.left - margin.right);
	let innerChartHeight: number = $derived(chartHeight - margin.top - margin.bottom);
	let xProperty = $state<string>('weight');
	let yProperty = $state<string>('raceTime');
	let colorProperty = $state<string>('name');

	$effect(() => {
		d3.csv(
			'https://raw.githubusercontent.com/gckirchoff/league-of-pigs-data/refs/heads/master/data.csv',
			(row) => {
				return {
					...row,
					height: Number(row.height),
					season: Number(row.season),
					round: Number(row.round),
					race: Number(row.race),
					length: Number(row.length),
					waist: Number(row.waist),
					weight: Number(row.weight),
					isMale: row['is male'] === 'TRUE',
					position: row.position,
					raceTime: Number(row['race time'])
				};
			}
		).then((result) => {
			data = result;
		});
	});

	let xScale = $derived(
		d3
			.scaleLinear()
			.domain(d3.extent(data, (row) => row[xProperty]) as [number, number])
			.range([0, innerChartWidth])
	);

	let xTicks = $derived(xScale.ticks());

	let yScale = $derived(
		d3
			.scaleLinear()
			.domain(d3.extent(data, (row) => row[yProperty]) as [number, number])
			.range([innerChartHeight, 0])
	);

	let yTicks = $derived(yScale.ticks());

	let colorScale = $derived(
		d3
			.scaleOrdinal<string, string>()
			.domain(Array.from(new Set(data.map((row) => row[colorProperty]))))
			.range(d3.schemeCategory10)
	);

	$inspect(data);
</script>

<div class="flex h-svh w-full flex-col items-center justify-center" bind:clientWidth={chartWidth}>
	<div class="bg-slate-200 p-2">
		<label for="x-prop">X Property</label>
		<select id="x-prop" bind:value={xProperty}>
			{#each data.length > 0 ? Object.keys(data[0]).filter((key) => typeof data[0][key] === 'number') : [] as value}
				<option {value}>{value}</option>
			{/each}
		</select>
		<label for="y-prop">Y Property</label>
		<select id="y-prop" bind:value={yProperty}>
			{#each data.length > 0 ? Object.keys(data[0]).filter((key) => typeof data[0][key] === 'number') : [] as value}
				<option {value}>{value}</option>
			{/each}
		</select>
	</div>
	<svg class="bg-white" width={chartWidth} height={chartHeight}>
		<g style="transform: translate({margin.left}px, {margin.top}px)">
			<g style="transform: translate(0, {innerChartHeight}px)">
				<line x1="0" y1="0" x2={innerChartWidth} y2="0" stroke="black" stroke-width="4px" />
				{#each xTicks as tick}
					<text y={25} x={xScale(tick)} dominant-baseline="middle" text-anchor="middle">{tick}</text
					>
				{/each}
				<text dominant-baseline="middle" text-anchor="middle" x={innerChartWidth / 2} y={50}
					>{xProperty}</text
				>
			</g>
			<g>
				<line x1="0" y1="0" x2="0" y2={innerChartHeight} stroke="black" stroke-width="4px" />
				{#each yTicks as tick}
					<text y={yScale(tick)} dominant-baseline="middle" text-anchor="middle" x={-25}
						>{tick}</text
					>
				{/each}
				<text
					dominant-baseline="middle"
					text-anchor="middle"
					class="rotate-270 transform"
					y={-50}
					x={-innerChartHeight / 2}>{yProperty}</text
				>
			</g>
			{#each data as row}
				<circle
					r="8px"
					fill={colorScale(row[colorProperty])}
					cy={yScale(row[yProperty])}
					cx={xScale(row[xProperty])}
					class="transition-all duration-500 ease-in"
				/>
			{/each}
		</g>
	</svg>
</div>
