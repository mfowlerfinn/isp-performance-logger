<script>
import { data } from "./Firebase.svelte";
import { onMount } from "svelte";

let dDown = [];
let dUp =[];
let dPing = [];
let dEpoch = [];
let isMounted = false;

onMount(() => {
  isMounted = true;
});

$: if($data.length > 1 && isMounted) {
  const renderCharts = () => {
      Plotly.newPlot('chart-plotly', dataToChart, layout, {displayModeBar: false, responsive: true});
  };

  let len = $data.length;

  for ( let i = 0; i < len; i++ ) {
    let sample = $data[i];
    let epoch = sample.timeStamp;
    //let epoch = new Date(sample.timeStamp).valueOf();
    dEpoch.push(epoch);
    dDown.push(sample.download);
    dUp.push(sample.upload);
    dPing.push(sample.ping);
  }

  renderCharts();
}



var layout = {
  xaxis: {
    showticklabels: false,
    showgrid: false
    // ticks: 'outside',
    // tick0: 0,
    // dtick: 5,
    // ticklen: 8,
    // tickwidth: 4,
    // tickcolor: '#000'
  },
  yaxis: {
    autotick: false,
    ticks: 'outside',
    tick0: 0,
    dtick: 10,
    ticklen: 4,
    tickwidth: 2,
    tickcolor: '#000'
  },
  // showlegend: true,
  font: {size: 18},
  legend: {
    orientation: "h"
    },
  margin: {
    l: 50,
    r: 50,
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
</script>