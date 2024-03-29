<template>
  <div v-if="!joined" class="parent-container">
    <div class="name-container">
      <input type="text" class="user-name" v-model="currentUser" />
      <button class="join-button" @click="join">Join</button>
    </div>
  </div>

  <div v-if="joined">
    <div class="list-container">
      <div v-for="message in messages" :key="message.id">
        <b>
          {{ message.user }}
        </b>
        : {{ message.text }}
      </div>
    </div>

    <div class="text-input-container">
      <textarea
        v-model="text"
        class="text-message"
        @keyup.enter="sendMessage"
      ></textarea>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";
import io, { Socket } from "socket.io-client";
import { v4 as uuidv4 } from "uuid";
import { Message } from "@/types/chat";

const joined = ref(false);
const currentUser = ref("");
const text = ref("");
const messages = ref<Message[]>([]);
const socketInstance = ref<Socket>();

function join(): void {
  joined.value = true;
  socketInstance.value = io("http://localhost:3000");
  socketInstance.value.on("message:received", (data) => {
    messages.value = messages.value.concat(data);
  });
}

function sendMessage(): void {
  addMessage();
  text.value = "";
}

function addMessage(): void {
  const message: Message = {
    id: uuidv4(),
    text: text.value,
    user: currentUser.value,
  };
  messages.value = messages.value.concat(message);
  socketInstance.value?.emit("message", message);
}
</script>

<style scoped>
.parent-container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  position: fixed;
  padding-top: 150px;
}

.name-container {
  display: flex;
  flex-direction: column;
  width: 200px;
}

.user-name {
  height: 30px;
  font-size: 20px;
  padding: 5px;
  margin-bottom: 5px;
  text-align: center;
  font-weight: bold;
}

.join-button {
  height: 30px;
  font-size: 20px;
}

.text-input-container {
  height: 100vh;
}

.text-message {
  width: 100%;
  position: absolute;
  bottom: 0px;
  height: 70px;
  padding: 10px;
  box-sizing: border-box;
}
</style>
