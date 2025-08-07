<script setup>
import { ref, onMounted, nextTick, computed, onUnmounted } from 'vue'
// import { io } from 'socket.io-client'

const socket = ref(null)
const connectionStatus = ref('disconnected')
const isConnected = computed(() => connectionStatus.value === 'connected')

const messages = ref([])
const messageToSend = ref('')

const SERVER_URL = 'wss://bluewater-tech-expo-2025-pman-sockets.onrender.com'

const isLooping = ref(true)

const pressed = ref(new Array(4).fill(0))

// Disconnect from WebSocket
const disconnect = () => {
  if (socket.value) {
    socket.value.close()
    socket.value = null
  }
}

// Connect to WebSocket
const connect = () => {
  try {
    connectionStatus.value = 'connecting'
    socket.value = new WebSocket(SERVER_URL)

    socket.value.onopen = () => {
      connectionStatus.value = 'connected'
      addMessage('Connected to server', 'system')
    }

    // socket.value.onmessage = (event) => {
    //   addMessage(event.data, 'received')
    // }

    socket.value.onclose = () => {
      connectionStatus.value = 'disconnected'
      addMessage('Disconnected from server', 'system')
    }

    socket.value.onerror = (error) => {
      connectionStatus.value = 'error'
      addMessage(`Connection error: ${error}`, 'error')
    }
  } catch (error) {
    connectionStatus.value = 'error'
    addMessage(`Failed to connect: ${error.message}`, 'error')
  }
}

const addMessage = (content, type = 'received') => {
  messages.value.push({
    content,
    type,
    timestamp: new Date().toLocaleTimeString(),
  })
}

function sleep(ms) {
  return new Promise((resolve) => setTimeout(resolve, ms))
}

function touchStart(e) {
  e.preventDefault()
  console.log(e.target.id)
  const index = parseInt(e.target.id, 10)
  console.log(index)
  pressed.value[index] = 1
}

function touchEnd(e) {
  e.preventDefault()

  const index = parseInt(e.target.id, 10)
  console.log(index)
  pressed.value[index] = 0
}

// Send message
const sendMessage = () => {
  if (socket.value && messageToSend.value.trim()) {
    socket.value.send(messageToSend.value)
    addMessage(messageToSend.value, 'sent')
    messageToSend.value = ''
  }
}

async function frameUpdate() {
  while (isLooping) {
    if (isConnected.value) {
      console.log(pressed.value)
      socket.value.send(JSON.stringify({ directions: pressed.value }))
    }

    await sleep(1000 / 30)
  }
}

// Cleanup on component unmount
onUnmounted(() => {
  disconnect()
})

onMounted(async () => {
  connect()
  await nextTick()
  await frameUpdate()
})
</script>

<template>
  <div class="container">
    <div
      id="0"
      class="circle up"
      @touchstart="touchStart"
      @touchend="touchEnd"
      @mousedown="touchStart"
      @mouseup="touchend"
      @mouseleave="touchend"
    ></div>
    <div
      id="1"
      class="circle right"
      @touchstart="touchStart"
      @touchend="touchEnd"
      @mousedown="touchStart"
      @mouseup="touchend"
      @mouseleave="touchend"
    ></div>
    <div
      id="2"
      class="circle down"
      @touchstart="touchStart"
      @touchend="touchEnd"
      @mousedown="touchStart"
      @mouseup="touchend"
      @mouseleave="touchend"
    ></div>
    <div
      id="3"
      class="circle left"
      @touchstart="touchStart"
      @touchend="touchEnd"
      @mousedown="touchStart"
      @mouseup="touchend"
      @mouseleave="touchend"
    ></div>
  </div>
</template>

<style>
body,
html {
  height: 100vh;
  width: 100vw;
  margin: 0;
  padding: 0;
}

#app {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>

<style scoped>
.container {
  flex-grow: 1;
  background-color: gray;
  width: 100%;
  height: 100%;
  /* width: 100vh; */
  /* height: 100vw; */
  position: relative;
}

.circle {
  position: absolute;
  background-color: white;
  width: 128px;
  aspect-ratio: 1;
  border-radius: 50%;

  transform: translate(-50%, -50%);
}

.up {
  top: calc(50% - 128px);
  left: 50%;
}

.down {
  top: calc(50% + 128px);
  left: 50%;
}

.left {
  top: 50%;
  left: calc(50% - 128px);
}

.right {
  top: 50%;
  left: calc(50% + 128px);
}
</style>
