<script setup>

import { io } from 'socket.io-client';
import { onBeforeMount, ref } from 'vue';

const socket = io('http://localhost:3000');
const messages = ref([]);
const messageText = ref('');
const joined = ref(false);
const name = ref('');
const typingDisplay = ref('');

onBeforeMount(() => {

  socket.emit('findAllMessages', {}, (response) => {
    console.log("[+] Messages recieved: ", response);
    messages.value = response;
  });

  socket.on('message', (message) => {
    messages.value.push(message);
  });

  socket.on('typing', (response) => {
    if(response.isTyping){
      typingDisplay.value = `${response.name} is typing...`;
    } else {
      typingDisplay.value = '';
    }
  });

});




const join = () => {
  socket.emit('join', { name: name.value}, () => {
    joined.value = true;
  })
}
const sendMessage = () => {
  console.log("[+] Sending message: ", messageText.value);
  socket.emit('createMessage', { text: messageText.value }, (message) => {
    messageText.value = '';
  });
}
let timeout;
const emitTyping = () => {
  socket.emit('typing', {isTyping: true});
  timeout = setTimeout(() => {
    socket.emit('typing', {isTyping: false});
  }, 2000);
}


</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label for="name">What's your name?</label>
        <input  id="name" v-model="name" />
        <button type="submit">Enter</button>
      </form>
    </div>

    <div v-else class="chat-container">
      <div class="messages-container">
        <div v-for="message in messages">
          [ {{ message.name }} ]: {{ message.text }}
        </div>
      </div>

      <div v-if="typingDisplay">{{ typingDisplay }}</div>
      <hr/>

      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label for="msg">message: </label>

          <input id="msg" v-model="messageText" @input="emitTyping" />
          <button type="submit">Send</button>
        </form>
      </div>

    </div>
  </div>
</template>

<style scoped>

</style>
