<template>
  <div class="examples-section">
    <h3>🤔 어떤 재료가 있나요?</h3>
    
      <!-- 입력된 재료 목록 -->
      <div class="ingredient-tags">
        <span
          v-for="(item, index) in ingredients"
          :key="index"
          class="tag"
        >
          {{ item }}
          <button class="remove-btn" @click="removeIngredient(index)">×</button>
        </span>
      </div>

      <!-- 재료 입력창 -->
      <div class="input-container">
        <input
          v-model="input"
          class="ingredient-input"
          placeholder="재료를 입력하세요 (Enter: 재료추가)"
          @keydown.enter.prevent="addIngredient"
        />
        <button @click="enterIngredients" class="send-btn">
          전송
        </button>
      </div>
    
  </div>
</template>

<script setup>
import { ref } from 'vue'

const messages = ref(['placeholder']) // 실제 메시지 로직에 맞게 대체하세요
const ingredients = ref([])
const input = ref('')

function addIngredient() {
  const trimmed = input.value.trim()
  if (trimmed && !ingredients.value.includes(trimmed)) {
    ingredients.value.push(trimmed)
    input.value = ''
  }
}

function removeIngredient(index) {
  ingredients.value.splice(index, 1)
}
const emit = defineEmits(['fillIngredients']);
function enterIngredients() {
  emit('fillIngredients', ingredients.value);
  ingredients.value = [];
}
</script>

<style scoped>
.ingredient-container {
  background: rgba(255, 255, 255, 0.9);
  padding: 1.5rem;
  border-radius: 15px;
  margin-bottom: 1rem;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
}

.ingredient-tags {
  margin-top: 16px;
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem;
  margin-bottom: 1rem;
}

.tag {
  background-color: #e0f0ff;
  color: #007acc;
  padding: 0.4rem 0.75rem;
  border-radius: 20px;
  font-size: 0.9rem;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
}

.remove-btn {
  background: none;
  border: none;
  font-weight: bold;
  color: #007acc;
  cursor: pointer;
  font-size: 1rem;
  padding: 0;
  line-height: 1;
}

.ingredient-input {
  width: 100%;
  padding: 0.75rem;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 10px;
}
.send-btn {
  background: linear-gradient(90deg, #88e2fb, #4658e3);
  color: #eee;
  border: none;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  justify-content: center;
}
.input-container{
  padding: 4px;
  display: flex;
  gap: 10px;
  justify-content: space-between;
}
</style>
