<script>
import { data } from "./Firebase.svelte";
import { onMount } from "svelte";

let dDown = [];
let dUp =[];
let dPing = [];
let dEpoch = [];
let isMounted = false;
let maxPing = 500;
$: maxSpeed = 0;

onMount(() => {
  isMounted = true;
});

$: if($data.length > 50 && isMounted) {
  let len = $data.length;

  for ( let i = 0; i < len; i++ ) {
    let sample = $data[i];
    //let epoch = sample.timeStamp;
    let epoch = new Date(sample.timeStamp).valueOf();
    dEpoch.push(epoch);
    dDown.push(sample.download);
    if(sample.download > maxSpeed) maxSpeed = sample.download;
    dUp.push(sample.upload);
    (sample.ping > maxPing) ? dPing.push(maxPing) : dPing.push(sample.ping);
  }

let now = Date.now();
let oneDayInSeconds = 86400000;
let lastDay = now - oneDayInSeconds;


var layout = {
  xaxis: {
    showticklabels: false,
    showgrid: true,
    ticks: 'outside',
    tick0: 4,
    dtick: 6, //6 tick per hour (one tick per sample)
    ticklen: 0,
    tickwidth: 0,
    type: 'date',
    range: [lastDay, now]
  },
  yaxis: {
    autotick: true,
    ticks: 'outside',
    tick0: 0,
    tickwidth: 2,
    tickcolor: '#000',
    range: [ 0, maxSpeed + (maxSpeed * 0.1) ]
  },
  // showlegend: true,
  font: {size: 18},
  legend: {
    orientation: "h"
  },
  margin: {
    l: 70,
    r: 70,
    b: 15,
    t: 15,
    pad: 4
  },
};

var lineUp = {
  name: "Upload (Mbps)",
  x: dEpoch,
  y: dUp,
  line: {shape: 'spline'},
  type: 'scatter'
};

var lineDown = {
  name: "Download (Mbps)",
  x: dEpoch,
  y: dDown,
  line: {shape: 'spline'},
  type: 'scatter'
};

var linePing = {
  name: "Ping (ms)",
  x: dEpoch,
  y: dPing,
  line: {
    shape: 'spline',
    dash: 'dot'
    },
  type: 'scatter'
};

var dataToChart = [lineDown, lineUp, linePing];

  const renderCharts = () => {
      Plotly.newPlot('chart-plotly', dataToChart, layout, {displayModeBar: true, responsive: true});
  };

  renderCharts();
}


</script>