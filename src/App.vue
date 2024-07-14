<template>
    <div class="left-pane">
      <CodeEditor v-model="program"/>
      <button @click="getSuggestions">Analyze</button>
    </div>
  <div>
    
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import CodeEditor from './components/CodeEditor.vue'


const program = ref(
`int main()
{
  int buf[32];
  int i = 45;

  buf[i] = 9;
}`
);


async function getSuggestions() {
  fetch('http://127.0.0.1:5000/analyze', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json'
    },
    body: JSON.stringify({ code: program })
  })
}

</script>

<style scoped>

.left-pane {
  width: 50%;
}

</style>
