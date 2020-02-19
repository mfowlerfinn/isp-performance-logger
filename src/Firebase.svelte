<script context="module">

import firebase from "firebase/app";
import "firebase/database";
import { writable } from "svelte/store";

export const data = writable("");
export const lastLine = writable("syncing with Firebase...");
export const upStats = writable("");
export const timeLeft = writable();
let block = [];
let countdown;

const config = {
  apiKey: "AIzaSyDEkrnseExa85HAHJAqe9SRW3CSoOxMmKY",
  authDomain: "bandwidth-logger.firebaseapp.com",
  databaseURL: "https://bandwidth-logger.firebaseio.com",
  projectId: "bandwidth-logger",
  storageBucket: "bandwidth-logger.appspot.com",
  messagingSenderId: "292907426099",
  appId: "1:292907426099:web:623a42e72b099b58fc9e46"
};
firebase.initializeApp(config);

const db = firebase.database();


let expectedSampleInterval = 600000; //in milliseconds (10 minutes)
let expectedTimeout = expectedSampleInterval / 3;
let maxInterval = expectedSampleInterval + expectedTimeout;
let numberSamples = 432; //last 3 days
let lastEpoch;

function getTime(epoch) {
    var formattedTime = new Date(epoch).toLocaleString([], {
      year: "numeric",
      month: "numeric",
      day: "numeric",
      hour: "2-digit",
      minute: "2-digit"
    });
    return formattedTime;
}

function sendData(t, stats) {
        console.log('downtime detected; updating database...');
        db.ref('speedTests/' + t).set(stats);
}

let averageSpeed;
let networkUp;
let percentUptime;

const getStats = (arr) => {
  let samples = arr.length;
  let sumDown = 0;
  let sumSpeed = 0;

  arr.map((sample, index) => {
    let {download, upload, ping, timeStamp} = sample;
    if (download === 0) {
      sumDown+=1;
    }
    sumSpeed+=download;
    if(index === (samples-1)) {
      if(download !== 0) {
        networkUp = true;
      }
      else {
        networkUp = false;
      }
    }
  });

  let percentDowntime = Number(((sumDown) / samples));
  percentUptime = Number(((1 - percentDowntime)*100).toFixed(2));
  averageSpeed = Number((sumSpeed / samples).toFixed(0));
  upStats.set({averageSpeed, networkUp, percentUptime});
}

var query = db.ref("speedTests/").orderByKey().limitToLast(numberSamples);

query.on("value", (snapshot) => {
  block = [];
  lastEpoch = 0;
  snapshot.forEach((childSnapshot) => {
    var epoch = parseInt(childSnapshot.key);
    while (lastEpoch + maxInterval < epoch && lastEpoch != 0) { //missing data, downtime assumed
      lastEpoch += expectedSampleInterval;
      let timeStamp = getTime(lastEpoch);
      let { download, ping, upload } = {download: 0, ping: 0, upload: 0};
      let t = lastEpoch;
      let stats = { download, upload, ping };
      sendData(t, stats);
      block.push({ timeStamp, download, ping, upload });
    } 

    let timeStamp = getTime(epoch);

    // childData will be the actual contents of the child
    var { download, ping, upload } = childSnapshot.val();
    block.push({ timeStamp, download, ping, upload });
    lastEpoch = epoch;
  });
  // console.log(block);
  let last = block[block.length - 1];
  lastLine.set(last);
  data.set("");
  getStats(block);
  data.set(block);
  let lastUpdate = Date.parse(last.timeStamp);
  clearInterval(countdown);
  timer(lastUpdate);

});

const timer = (lastUpdate) => {
  const sampleDelay = 20;
  const tenMin = (60 * 10) + sampleDelay;
  const now = Date.now();
  lastUpdate = (lastUpdate > tenMin + now) ? lastUpdate%600 : lastUpdate;
  const then = lastUpdate + tenMin * 1000;
  countdown = setInterval(() => {
    const secondsLeft = Math.round((then - Date.now()) / 1000);
    if (secondsLeft < -sampleDelay) {
      clearInterval(countdown);
      networkUp = false;
      upStats.set({averageSpeed, networkUp, percentUptime});
      return;
    }
    // console.log(secondsLeft);
    timeLeft.set(secondsLeft);
  }, 1000);
}

</script>
