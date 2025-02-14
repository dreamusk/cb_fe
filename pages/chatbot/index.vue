<template>
    <div>
        <div class="logo-container">
            <a href="">
                <img src="/logo.png" width="200" height="200" style="margin: 20px auto" />
            </a>
        </div>
        <div class="chat-box">
            <div class="chat-box-header">CACS ASSISTANT</div>
            <div class="chat-box-body" ref="chatBoxBody">
                <div v-for="(message, index) in messages" :key="index" :class="['message-container', message.type]">
                    <p>{{ message.content }}</p>
                </div>
                <div v-if="isLoading" class="loading-dots">
                    {{ loadingDots }}
                </div>
            </div>
            <div class="chat-box-footer">
                <input type="text" placeholder="Ask a question..." v-model="messageInput" @keypress.enter="sendMessage"
                    :disabled="isLoading" />
                <button @click="sendMessage" :disabled="isLoading">Send</button>
            </div>
        </div>
        <!-- <div class="Cacs">
            <a href="">Cacs system</a>
        </div> -->
    </div>
</template>

<script setup>
import { ref, watch, nextTick } from 'vue'

const chatBoxBody = ref(null)
const messageInput = ref('')
const messages = ref([
    { type: 'response', content: 'Welcome to CACS ASSISTANT, how can I help you today?' }
])
const isLoading = ref(false)
const loadingDots = ref('.')
let loadingInterval = null

const scrollToBottom = () => {
    if (chatBoxBody.value) {
        nextTick(() => {
            chatBoxBody.value.scrollTop = chatBoxBody.value.scrollHeight
        })
    }
}

const startLoadingAnimation = () => {
    loadingInterval = setInterval(() => {
        loadingDots.value = loadingDots.value.length >= 3
            ? '.'
            : loadingDots.value + '.'
    }, 250)
}

const stopLoadingAnimation = () => {
    if (loadingInterval) {
        clearInterval(loadingInterval)
        loadingInterval = null
        loadingDots.value = '.'
    }
}

const sendMessage = async () => {
    const message = messageInput.value.trim()
    if (!message || isLoading.value) return

    // Clear input and add user message
    messageInput.value = ''
    messages.value.push({ type: 'user', content: message })
    isLoading.value = true
    startLoadingAnimation()
    scrollToBottom()

    try {
        const response = await fetch('https://chatbot1-4ji0.onrender.com/api/predict', {
            method: 'POST',
            headers: {
                'Accept': 'application/json',
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({ message })
        })

        const data = await response.json()

        if (data.status === 'finished' && data.status_code === 200) {
            messages.value.push({
                type: 'response',
                content: data.result.answer
            })
        } else {
            throw new Error('Invalid response from server')
        }
    } catch (error) {
        console.error('Error:', error)
        messages.value.push({
            type: 'error',
            content: 'Sorry, there was an error processing your request.'
        })
    } finally {
        isLoading.value = false
        stopLoadingAnimation()
        scrollToBottom()
    }
}

// Watch messages and scroll to bottom when they change
watch(messages, () => {
    scrollToBottom()
}, { deep: true })

// Clean up on unmount
onUnmounted(() => {
    stopLoadingAnimation()
})
</script>

<style lang="scss" scoped>
body {
    font-family: 'Roboto', sans-serif;
    background: #010141;
    background-size: cover;
    background-position: center center;
}

.logo-container {
    width: 100%;
    text-align: center;
}

.chat-box {
    max-width: 800px;
    margin: 0 auto;
    border: 1px solid #ddd;
    border-radius: 8px;

    .chat-box-header {
        padding: 25px;
        background: #f8f9fa;
        border-bottom: 1px solid #ddd;
        font-weight: bold;
        font-size: 30px;
    }

    .chat-box-body {
        height: 400px;
        overflow-y: auto;
        padding: 15px;
    }

    .message-container {
        margin: 10px;
        padding: 10px;
        border-radius: 8px;

        &.user {
            background-color: #e3f2fd;
            margin-left: 20%;
        }

        &.response {
            background-color: #f5f5f5;
            margin-right: 20%;
        }

        &.error {
            background-color: #ffebee;
            color: #c62828;
        }
    }

    .loading-dots {
        padding: 10px;
        color: #666;
        font-size: 50px;
    }

    .chat-box-footer {
        padding: 20px;
        border-top: 1px solid #ddd;
        display: flex;
        gap: 10px;

        input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 4px;
        }

        button {
            padding: 10px 16px;
            background: #007bff;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;

            &:disabled {
                background: #ccc;
            }
        }
    }
}
</style>