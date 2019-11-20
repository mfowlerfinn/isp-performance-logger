<script>
	import { data, lastLine } from './Firebase.svelte';
	import Charts from './Charts.svelte';

	export let name;
	$: objString = JSON.stringify($data);
	$: last = `Last Updated at ${$lastLine.timeStamp}`;
</script>

<svelte:head>
	<title>{name}</title>
	<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bulma@0.8.0/css/bulma.css" />
</svelte:head>

<style>
	h1 {
		color: purple;
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