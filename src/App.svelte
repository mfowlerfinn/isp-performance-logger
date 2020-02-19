<script>
	import { data, lastLine, upStats, timeLeft } from './Firebase.svelte';
	import Charts from './Charts.svelte';

	let averageSpeed, networkUp, percentUptime;

	export let name;
	$: objString = JSON.stringify($data);
	$: last = ($lastLine.timeStamp) ? `Updated at ${$lastLine.timeStamp}` : `${$lastLine}`;
	$: averageSpeed = $upStats.averageSpeed;
	$: networkUp = $upStats.networkUp;
	$: percentUptime = $upStats.percentUptime;
</script>
<Charts />

<svelte:head>
	<title>{name}</title>
</svelte:head>


<main class="content">
	
	{#if $upStats}
		<div id="title-container"  networkUp={networkUp}>
			<h1>{name}</h1>
			{#if (networkUp === true)}
			<div class="stat">{`Network UP, averaging ${averageSpeed}Mbps`}</div>
			{:else}
			<div class="stat">Network DOWN!</div>
			{/if}
			<div class="stat">{`${percentUptime}% uptime`}</div>
		</div>
		{:else}
		<div id="title-container">
			<h1>{name}</h1>
			<h2>syncing with Firebase...</h2>
		</div>
	{/if}
	<div id="chart-plotly"></div>
	<footer id="update-status">
		<div class="update-text">{last}</div>
		{#if $timeLeft}<div class="update-text">{`Update expected in ${$timeLeft}s`}</div>{/if}
	</footer>

</main>

