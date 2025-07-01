<!-- App.vue -->
<script setup>
import { ref, onMounted, nextTick } from 'vue'
import Ingredient from './components/ingredient.vue'

// 반응형 상태
const messages = ref([])
const userInput = ref('')
const isLoading = ref(false)
const conversationId = ref(null)
const messagesContainer = ref(null)

// API 기본 URL (개발 환경)
const API_BASE_URL = 'http://127.0.0.1:8000'

// 초기 메시지
onMounted(() => {
  messages.value.push({
    type: 'bot',
    content: '안녕하세요! 🍳 한국 요리 레시피 도우미입니다.\n궁금한 요리나 레시피에 대해 무엇이든 물어보세요!',
    timestamp: new Date().toLocaleTimeString()
  })
})

// 메시지 포맷팅 함수 (마크다운 기호만 제거)
const formatMessage = (content) => {
  if (!content) return content
  
  // 간단하게 마크다운 기호들만 제거
  return content
    .replace(/\*\*/g, '')  // ** 제거
    .replace(/###/g, '')   // ### 제거
    .replace(/##/g, '')    // ## 제거
    .replace(/#/g, '')     // # 제거
}

// 메시지 전송 함수
const sendMessage = async () => {
  if (!userInput.value.trim() || isLoading.value) return

  const userMessage = userInput.value.trim()
  userInput.value = ''

  // 사용자 메시지 추가
  messages.value.push({
    type: 'user',
    content: userMessage,
    timestamp: new Date().toLocaleTimeString()
  })

  // 로딩 상태 설정
  isLoading.value = true

  try {
    // API 요청
    const response = await fetch(`${API_BASE_URL}/chat`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        message: userMessage,
        conversation_id: conversationId.value
      })
    })

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`)
    }

    const data = await response.json()

    // 대화 ID 저장 (첫 번째 응답에서)
    if (!conversationId.value && data.conversation_id) {
      conversationId.value = data.conversation_id
    }

    // 봇 응답 추가
    messages.value.push({
      type: 'bot',
      content: data.response,
      timestamp: new Date().toLocaleTimeString(),
      sources: data.sources,
      responseTime: data.response_time
    })

  } catch (error) {
    console.error('Error:', error)
    messages.value.push({
      type: 'bot',
      content: '죄송합니다. 서버에 연결할 수 없습니다. 잠시 후 다시 시도해주세요.',
      timestamp: new Date().toLocaleTimeString(),
      isError: true
    })
  } finally {
    isLoading.value = false
    // 스크롤을 맨 아래로
    await nextTick()
    scrollToBottom()
  }
}

// 스크롤을 맨 아래로 이동
const scrollToBottom = () => {
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight
  }
}

// Enter 키 처리
const handleKeyPress = (event) => {
  if (event.key === 'Enter' && !event.shiftKey) {
    event.preventDefault()
    sendMessage()
  }
}

// 대화 초기화
const clearChat = () => {
  messages.value = [{
    type: 'bot',
    content: '안녕하세요! 🍳 한국 요리 레시피 도우미입니다.\n궁금한 요리나 레시피에 대해 무엇이든 물어보세요!',
    timestamp: new Date().toLocaleTimeString()
  }]
  conversationId.value = null
}

// 예시 질문들
const exampleQuestions = [
  '김치찌개 만드는 법 알려줘',
  '간단한 볶음밥 레시피',
  '초보자도 할 수 있는 요리',
  '닭가슴살 요리법'
]

// 예시 질문 클릭 처리
const askExample = (question) => {
  userInput.value = question
  sendMessage()
}

const onIngredientsInput = (ingredients) => {
  userInput.value = ingredients.join(',');
  sendMessage()
}
</script>

<template>
  <div class="chat-app">
    <!-- 헤더 -->
    <header class="chat-header">
      <div class="header-content">
        <h1>🍳 한국 요리 레시피 챗봇</h1>
        <button @click="clearChat" class="clear-btn">
          🗑️ 대화 초기화
        </button>
      </div>
    </header>

    <!-- 메시지 영역 -->
    <div class="messages-container" ref="messagesContainer">
      <div class="messages">
        <!-- 예시 질문 (첫 번째 메시지 후에만 표시) -->
        <div v-if="messages.length === 1" class="examples-section">
          <h3>💡 이런 질문을 해보세요:</h3>
          <div class="example-buttons">
            <button 
              v-for="question in exampleQuestions" 
              :key="question"
              @click="askExample(question)"
              class="example-btn"
            >
              {{ question }}
            </button>
          </div>
        </div>
        <Ingredient @fill-ingredients="onIngredientsInput"/>
        <!-- 메시지들 -->
        <div 
          v-for="(message, index) in messages" 
          :key="index" 
          class="message"
          :class="message.type"
        >
          <div class="message-content">
            <div class="message-bubble">
              <pre>{{ formatMessage(message.content) }}</pre>
              
              <!-- 소스 정보 (봇 메시지인 경우) -->
              <div v-if="message.sources && message.sources.length > 0" class="sources">
                <h4>📚 참고 레시피:</h4>
                <div v-for="source in message.sources" :key="source.title" class="source-item">
                  <strong>{{ source.title }}</strong>
                  <span class="source-meta">
                    {{ source.category }} | {{ source.difficulty }} | {{ source.time }}
                  </span>
                </div>
              </div>
            </div>
            
            <div class="message-info">
              <span class="timestamp">{{ message.timestamp }}</span>
              <span v-if="message.responseTime" class="response-time">
                ({{ Math.round(message.responseTime * 1000) }}ms)
              </span>
            </div>
          </div>
        </div>

        <!-- 로딩 표시 -->
        <div v-if="isLoading" class="message bot">
          <div class="message-content">
            <div class="message-bubble loading">
              <div class="typing-indicator">
                <span></span>
                <span></span>
                <span></span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- 입력 영역 -->
    <div class="input-area">
      <div class="input-container">
        <textarea
          v-model="userInput"
          @keypress="handleKeyPress"
          placeholder="레시피에 대해 무엇이든 물어보세요..."
          rows="1"
          :disabled="isLoading"
          class="message-input"
        ></textarea>
        <button 
          @click="sendMessage" 
          :disabled="!userInput.trim() || isLoading"
          class="send-btn"
        >
          {{ isLoading ? '⏳' : '📤' }}
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.chat-app {
  min-width: 50vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  /* background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); */
  background: white;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.chat-header {
  /* background: rgba(255, 255, 255, 0.95); */
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%); 
  backdrop-filter: blur(10px);
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
  padding: 1rem;
  box-shadow: 0 2px 20px rgba(0, 0, 0, 0.1);
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  max-width: 800px;
  margin: 0 auto;
}

.chat-header h1 {
  margin: 0;
  color: #eee;
  font-size: 1.5rem;
  font-weight: 600;
}

.clear-btn {
  background: #ff6b6b;
  color: white;
  border: none;
  padding: 0.5rem 1rem;
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s ease;
}

.clear-btn:hover {
  background: #ff5252;
  transform: translateY(-1px);
}

.messages-container {
  flex: 1;
  overflow-y: auto;
  padding: 1rem;
  background: rgba(255, 255, 255, 0.05);
}

.messages {
  max-width: 800px;
  margin: 0 auto;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.examples-section {
  background: rgba(255, 255, 255, 0.9);
  padding: 1.5rem;
  border-radius: 15px;
  margin-bottom: 1rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.examples-section h3 {
  margin: 0 0 1rem 0;
  color: #333;
  font-size: 1.1rem;
}

.example-buttons {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 0.5rem;
}

.example-btn {
  background: linear-gradient(45deg, #667eea, #764ba2);
  color: white;
  border: none;
  padding: 0.75rem 1rem;
  border-radius: 10px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.2s ease;
  text-align: left;
}

.example-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.2);
}

.message {
  display: flex;
  margin-bottom: 1rem;
}

.message.user {
  justify-content: flex-end;
}

.message.bot {
  justify-content: flex-start;
}

.message-content {
  max-width: 70%;
  display: flex;
  flex-direction: column;
  gap: 0.25rem;
}

.message-bubble {
  padding: 1rem 1.25rem;
  border-radius: 20px;
  position: relative;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.user .message-bubble {
  background: linear-gradient(45deg, #667eea, #764ba2);
  color: #eee;
  border-bottom-right-radius: 5px;
}

.bot .message-bubble {
  background: rgba(255, 255, 255, 0.95);
  color: #333;
  border-bottom-left-radius: 5px;
}

.message-bubble pre {
  white-space: pre-wrap;
  word-break: break-word;
  margin: 0;
  font-family: inherit;
  line-height: 1.5;
}

.sources {
  margin-top: 1rem;
  padding-top: 1rem;
  border-top: 1px solid rgba(0, 0, 0, 0.1);
}

.sources h4 {
  margin: 0 0 0.5rem 0;
  font-size: 0.9rem;
  color: #666;
}

.source-item {
  display: flex;
  flex-direction: column;
  margin-bottom: 0.5rem;
  padding: 0.5rem;
  background: rgba(0, 0, 0, 0.05);
  border-radius: 8px;
}

.source-item strong {
  font-size: 0.9rem;
  color: #333;
}

.source-meta {
  font-size: 0.8rem;
  color: #666;
  margin-top: 0.25rem;
}

.message-info {
  display: flex;
  gap: 0.5rem;
  align-items: center;
  font-size: 0.75rem;
  color: rgba(255, 255, 255, 0.7);
}

.user .message-info {
  justify-content: flex-end;
  color: rgba(0, 0, 0, 0.5);
}

.bot .message-info {
  justify-content: flex-start;
  color: rgba(0, 0, 0, 0.5);
}

.loading .message-bubble {
  padding: 1rem 1.5rem;
}

.typing-indicator {
  display: flex;
  gap: 4px;
  align-items: center;
}

.typing-indicator span {
  width: 8px;
  height: 8px;
  border-radius: 50%;
  background: #999;
  animation: typing 1.4s infinite ease-in-out;
}

.typing-indicator span:nth-child(1) { animation-delay: -0.32s; }
.typing-indicator span:nth-child(2) { animation-delay: -0.16s; }

@keyframes typing {
  0%, 80%, 100% { transform: scale(0.8); opacity: 0.5; }
  40% { transform: scale(1); opacity: 1; }
}

.input-area {
  background: rgba(255, 255, 255, 0.95);
  backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.2);
  padding: 1rem;
}

.input-container {
  max-width: 800px;
  margin: 0 auto;
  display: flex;
  gap: 1rem;
  align-items: flex-end;
}

.message-input {
  flex: 1;
  padding: 1rem 1.25rem;
  border: 2px solid rgba(102, 126, 234, 0.2);
  border-radius: 20px;
  resize: none;
  font-family: inherit;
  font-size: 1rem;
  line-height: 1.5;
  background: white;
  transition: all 0.2s ease;
  min-height: 50px;
  max-height: 150px;
}

.message-input:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.message-input:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

.send-btn {
  background: linear-gradient(45deg, #667eea, #764ba2);
  color: white;
  border: none;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
  font-size: 1.2rem;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}

.send-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 15px rgba(102, 126, 234, 0.3);
}

.send-btn:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

/* 스크롤바 스타일링 */
.messages-container::-webkit-scrollbar {
  width: 6px;
}

.messages-container::-webkit-scrollbar-track {
  background: rgba(255, 255, 255, 0.1);
  border-radius: 3px;
}

.messages-container::-webkit-scrollbar-thumb {
  background: rgba(255, 255, 255, 0.3);
  border-radius: 3px;
}

.messages-container::-webkit-scrollbar-thumb:hover {
  background: rgba(255, 255, 255, 0.5);
}

/* 반응형 디자인 */
@media (max-width: 768px) {
  .chat-header h1 {
    font-size: 1.2rem;
  }
  
  .message-content {
    max-width: 85%;
  }
  
  .input-container {
    padding: 0;
  }
  
  .example-buttons {
    grid-template-columns: 1fr;
  }
}
</style>