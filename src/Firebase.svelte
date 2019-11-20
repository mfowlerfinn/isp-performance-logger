<script context="module">

import firebase from "firebase/app";
import "firebase/database";
import { writable } from "svelte/store";
// import { createEventDispatcher } from "svelte";
// const dispatch = createEventDispatcher();

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

var query = firebase
  .database()
  .ref("speedTests/")
  .orderByKey();
query.once("value").then(function(snapshot) {
  snapshot.forEach(function(childSnapshot) {
    var epoch = parseInt(childSnapshot.key);
    var timeStamp = new Date(epoch).toLocaleString([], {
      year: "numeric",
      month: "numeric",
      day: "numeric",
      hour: "2-digit",
      minute: "2-digit"
    });
    // childData will be the actual contents of the child
    var { download, ping, upload } = childSnapshot.val();
    block.push({ timeStamp, download, ping, upload });
  });
  console.log(block);
  let last = block[block.length - 1];
  lastLine.set(last);
  data.set(block);
  //data is ready
  // dispatch('ready');
});
</script>



<p>test</p>
