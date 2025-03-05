<script setup lang="ts">
import axios from 'axios';
import { nextTick, onMounted, ref } from 'vue';

const userInput = ref('')
const messages = ref<Array<{ text: string; type: 'user' | 'bot' }>>([])
const messageContainer = ref<HTMLElement | null>(null)

const scrollToBottom = async () => {
  await nextTick()
  if (messageContainer.value) {
    messageContainer.value.scrollTop = messageContainer.value.scrollHeight
  }
}

const sendMessage = async () => {
  if (!userInput.value.trim()) return

  // Add user message
  messages.value.push({
    text: userInput.value,
    type: 'user'
  })

  const userMessage = userInput.value
  userInput.value = ''

  try {
    // Make API call to your chatbot backend
    const response = await axios.post('/api/chat', {
      message: userMessage
    }, {
      headers: {
        'Content-Type': 'application/json'
      }
    })

    // Add bot response
    messages.value.push({
      text: response.data.reply,
      type: 'bot'
    })

    scrollToBottom()
  } catch (error) {
    console.error('Error sending message:', error)
    messages.value.push({
      text: 'Sorry, I encountered an error. Please try again.',
      type: 'bot'
    })
  }
}

onMounted(() => {
  // Initial greeting message
  messages.value.push({
    text: 'Hello! How can I help you today?',
    type: 'bot'
  })
})
</script>

<template>
  <VCard class="chatbot-container" elevation="0">
    <!-- Chat Header -->
    <VCardTitle class="chat-header pa-4">
      <VIcon icon="mdi-robot" class="mr-2" />
      Chat Support
    </VCardTitle>

    <VDivider />

    <!-- Messages Area -->
    <VCardText class="chat-messages pa-4" ref="messageContainer">
      <div v-for="(message, index) in messages" :key="index" class="message-wrapper" :class="message.type">
        <div class="message-bubble" :class="message.type">
          <v-icon v-if="message.type === 'bot'" icon="mdi-robot" size="small" class="mr-2" color="primary" />
          {{ message.text }}
        </div>
      </div>
    </VCardText>

    <VDivider />

    <!-- Input Area -->
    <VCardActions class="chat-input pa-4">
      <v-text-field v-model="userInput" density="comfortable" variant="outlined" placeholder="Type your message..."
        hide-details @keyup.enter="sendMessage" class="mr-2" bg-color="white">
        <template v-slot:append-inner>
          <VBtn icon color="primary" size="small" @click="sendMessage" :disabled="!userInput.trim()" variant="flat">
            <v-icon>mdi-send</v-icon>
          </VBtn>
        </template>
      </v-text-field>
    </VCardActions>
  </VCard>
</template>

<style scoped>
.chatbot-container {
  display: flex;
  overflow: hidden;
  flex-direction: column;
  background-color: #f5f5f7;
  block-size: 500px;
  inline-size: 350px;
}

.chat-header {
  background: rgb(var(--v-theme-primary));
  color: white;
}

.chat-messages {
  display: flex;
  flex: 1;
  flex-direction: column;
  padding: 16px;
  background-color: #f5f5f7;
  gap: 12px;
  overflow-y: auto;
}

.message-wrapper {
  display: flex;
  inline-size: 100%;
  margin-block: 4px;
  margin-inline: 0;
}

.message-wrapper.user {
  justify-content: flex-end;
}

.message-wrapper.bot {
  justify-content: flex-start;
}

.message-bubble {
  display: flex;
  align-items: center;
  border-radius: 12px;
  max-inline-size: 80%;
  padding-block: 8px;
  padding-inline: 12px;
}

.message-bubble.user {
  border-radius: 12px 12px 0;
  background-color: rgb(var(--v-theme-primary));
  color: white;
}

.message-bubble.bot {
  border-radius: 12px 12px 12px 0;
  background-color: white;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 5%);
  color: #2d3748;
}

.chat-input {
  background-color: #f5f5f7;
  border-block-start: 1px solid rgba(var(--v-border-color), var(--v-border-opacity));
}

/* Custom Scrollbar */
.chat-messages::-webkit-scrollbar {
  inline-size: 6px;
}

.chat-messages::-webkit-scrollbar-track {
  background: transparent;
}

.chat-messages::-webkit-scrollbar-thumb {
  border-radius: 3px;
  background-color: rgba(0, 0, 0, 10%);
}

.chat-messages::-webkit-scrollbar-thumb:hover {
  background-color: rgba(0, 0, 0, 20%);
}

/* Style for the input field */
:deep(.v-field) {
  border-radius: 8px;
  background-color: white !important;
}

/* Optional: Add transition for smooth interactions */
.message-bubble {
  transition: all 0.2s ease;
}

.message-wrapper {
  animation: fadeIn 0.3s ease;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
