<template>
  <h1>AST Insight</h1>
  <div class="content-wrapper">
    <div class="left-pane">
      <CodeEditor v-model="program"/>
      <button @click="getSuggestions">Analyze</button>
    </div>
    <div class="right-pane">
      <div v-if="loading">Loading...</div>
      <div v-else-if="error">
        <div>Something went wrong...</div> 
        <div>Stack trace:</div> 
        <CodeEditor :height="250" disabled v-model="error"/>
      </div>
      
      <div class="suggestion-wrapper" v-else-if="suggestions">
        <div v-for="suggestion of suggestions">
          <CodeEditor :height="150" disabled v-model="suggestion.code"/>
          <div>{{ suggestion.description }}</div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import CodeEditor from './components/CodeEditor.vue'

interface Suggestion {
  code: string;
  description: string;
}

const program = ref(
`int main()
{
  int buf[32];
  int i = 45;
  buf[i] = 9;
}`
);

const loading = ref(false);
const error = ref('');
const suggestions = ref<Suggestion[]>([]);


function getSuggestions() {
  loading.value = true;
  const url = 'http://127.0.0.1:5000/analyze';
  fetch(url, {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ code: program.value })
  }).then(async (response) => {
    const json = await response.json();
    if(response.ok){
      suggestions.value = json;
      error.value = '';
    } else {
      error.value = json.error;
    }
    loading.value = false;
  });
}

</script>

<style scoped>

.content-wrapper {
  display: flex;
  gap: 12px;
}

.left-pane {
  width: 40%;
}

.right-pane {
  max-width: 40%;
}

.suggestion-wrapper {
  display: flex;
  flex-direction: column;
  gap: 12px;
}

</style>
