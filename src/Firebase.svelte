<script context="module">

import firebase from "firebase/app";
import "firebase/database";
import { writable } from "svelte/store";

export const data = writable("syncing with Firebase...");
export const lastLine = writable("");

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

let block = [];

let expectedSampleInterval = 600000; //10 minutes
let expectedTimeout = expectedSampleInterval / 3;
let maxInterval = expectedSampleInterval + expectedTimeout;
let lastEpoch = 0;

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


var query = db.ref("speedTests/").orderByKey().limitToLast(432); //last 3 days

query.once("value").then(function(snapshot) {
  snapshot.forEach(function(childSnapshot) {
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
  data.set(block);
  //data is ready
  // dispatch('ready');
});


</script>



<p>test</p>
