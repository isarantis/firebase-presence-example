<script setup>
import { ref } from 'vue'
import { initializeApp } from 'firebase/app';
import { getDatabase,ref as fbRef,set as fbSet, onValue, serverTimestamp, onDisconnect, remove } from "firebase/database";
import { firebaseConfig } from "./boot/firebaseConfig.ts"

const loggedIn = ref(false)
const username= ref("")

const app = initializeApp(firebaseConfig); //intialize app
const database = getDatabase(); // select database

const onlineUsers = ref([])//vue users
const onlineUsersRef = fbRef(database, 'online') //online users FIREBASE

onValue(onlineUsersRef, (snapshot) => {
  const data = snapshot.val();
  onlineUsers.value = data
});

let disconnectRef = null

const loginUser = async () => {
  console.log('login',username.value)
  const userRef = fbRef(database, 'online/' + username.value)
  
  fbSet(userRef, {
    name: username.value,
    status: "✔️ Online",
    date:  serverTimestamp()
  });

  if(disconnectRef){
    console.log('login cleared')
    disconnectRef.cancel()
  }

  disconnectRef = onDisconnect(userRef)
  disconnectRef.set({
    name: username.value,
    status: "❌ Offline",
    date:  serverTimestamp()
  })


  loggedIn.value = true;
}

const logoutUser = async () => {
  console.log('logout',username.value)


  const userRef = fbRef(database, 'online/' + username.value)
  fbSet(userRef, {
    name: username.value,
    status: "❌ Offline",
    date:  serverTimestamp()
  });

  if(disconnectRef){
    console.log('logoutUser cleared')
    disconnectRef.cancel()
  }

  loggedIn.vale = false;
}

const deleteUser = async () => {
  console.log('delete',username.value)
  const userRef = fbRef(database, 'online/' + username.value)
  remove(userRef);

  if(disconnectRef){
    console.log('deleteUser cleared')
    disconnectRef.cancel()
  }
}

</script>

<template>
   <h1>Firebase Presence Demo</h1>

<div class="card">
  <input v-model="username" :disabled="loggedIn"/>
  <button type="button" @click="loginUser">login</button>
  <button type="button" @click="logoutUser">logout</button>
  <button type="button" @click="deleteUser">delete</button>
  

  <h3>Online users</h3>
  <p v-for="user in onlineUsers" v-bind:key="user.name">
    <strong>{{ user.name }}</strong> is <code>{{user.status}}</code> at {{ new Date(user.date) }}
  </p>
</div>
</template>