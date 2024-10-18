<script setup>
import { io } from 'socket.io-client';
import { onBeforeMount, ref } from 'vue';

const socket = io('http://localhost:3001/');
const messages = ref([]);
const messageText = ref('');
const joined = ref(false);
const name = ref('');
const typingDisplay = ref('');

onBeforeMount(() => {
  socket.emit('findAllMessage', {}, (response) => {
    messages.value = response;
  });

  socket.on('message', (message) => {
    messages.value.push(message);
  });

  socket.on('typing', ({ name, isTyping }) => {
    typingDisplay.value = isTyping ? `${name} is typing...` : '';
  });
});

const join = () => {
  socket.emit('join', { name: name.value, clientId: socket.id }, () => {
    joined.value = true;
  });
};

const sendMessage = () => {
  socket.emit('createMessage', { text: messageText.value, clientId: socket.id }, (response) => {
    messages.value.push(response);
  });
};

let timeout;
const emitTyping = () => {
  clearTimeout(timeout);
  socket.emit('typing', { isTyping: true });
  timeout = setTimeout(() => {
    socket.emit('typing', { isTyping: false });
  }, 1000);
};
</script>

<template>
  <div class="chat">
    <div v-if="!joined">
      <form @submit.prevent="join">
        <label>Enter your name please:</label>
        <input v-model="name" type="text">
        <button type="submit">Join</button>
      </form>
    </div>

    <div class="chat-container">
      <div class="message-container">
        <div v-for="message in messages" :key="message.id">
            [{{ message.name }}]: {{ message.text }}
        </div>
      </div>
      <div v-if="typingDisplay">{{ typingDisplay }}</div>
      <hr/>
      <div class="message-input">
        <form @submit.prevent="sendMessage">
          <label>Message:</label>
          <input v-model="messageText" @input="emitTyping" type="text">
          <button type="submit">Send</button>
        </form>
      </div>
    </div>
  </div>
</template>

<style>
.chat {
  padding: 20px;
  height: 100vh;
}
.chat-container {
  display: flex;
  flex-direction: column;
  height: 100%;
}
.message-container {
  flex: 1;
}
</style>