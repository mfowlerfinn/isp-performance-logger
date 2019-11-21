<script>
	import { data, lastLine } from './Firebase.svelte';
	import Charts from './Charts.svelte';

	export let name;
	$: objString = JSON.stringify($data);
	$: last = `Updated at ${$lastLine.timeStamp}`;
</script>

<svelte:head>
	<title>{name}</title>
</svelte:head>

<style>
	h1 {
		color: green;
	}
	main {
		margin: 20px;
	}
</style>

<main class="content">
	<h1>{name}</h1>
	<Charts />
	<div id="chart-plotly"></div>
	<p>{last}</p>
</main>