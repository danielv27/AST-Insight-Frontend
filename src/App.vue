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
        <div>{{ error }}</div> 
        <div v-if="errorTrace">
          <div>Stack trace:</div>
          <CodeEditor :height="252" disabled v-model="errorTrace"/>
        </div> 
      </div>
      
      <div class="suggestion-wrapper" v-else-if="suggestions">
        <div v-for="suggestion of suggestions">
          <CodeEditor :line-to-select="suggestion.line" :height="150" disabled v-model="suggestion.code"/>
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
  line: number;
}

const program = ref(
`#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void vulnerable_function() 
{
    char *buffer = malloc(12);
    for(int i = 0; i < 30; i++)
    {
        buffer[i - 5] = 'b';
    }
  
    char buffer2[16];
    int j = 0;
    while(j < 45){
        buffer2[j - 5] = 'b';
        j++;
    }

}

int main(int argc, char *argv[]) {
    vulnerable_function(argv[1]);
    return 0;
}`
);

const loading = ref(false);
const error = ref('');
const errorTrace = ref('');
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
  })
  .then(async (response) => {
    console.log(response.status)
    const json = await response.json();
    if(response.ok){
      suggestions.value = json;
      error.value = '';
      errorTrace.value = '';
    } else {
      error.value = 'Something went wrong...';
      errorTrace.value = json.error;
    }
    loading.value = false;
  })
  .catch((err) => {
    console.error('Network error:', err);
    error.value = 'A network error occurred. Please try again later.';
    errorTrace.value = '';
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
  display: flex;
  flex-direction: column;
  gap: 12px;
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
