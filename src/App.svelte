<script>
	import { data, lastLine } from './Firebase.svelte';
	import Charts from './Charts.svelte';

	export let name;
	$: objString = JSON.stringify($data);
	$: last = ($lastLine.timeStamp) ? `Updated at ${$lastLine.timeStamp}` : `Downloading from database...`;
</script>
<Charts />

<svelte:head>
	<title>{name}</title>
</svelte:head>


<main class="content">
	<h1>{name}</h1>
	<!-- <p id="update-status">{last}</p> -->
	<div id="chart-plotly"></div>
</main>