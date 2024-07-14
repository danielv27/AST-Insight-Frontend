<template>
    <Codemirror :extensions="extensions" v-model="model" />
</template>

<script setup lang="ts">
import { oneDark } from '@codemirror/theme-one-dark';
import { Codemirror } from 'vue-codemirror';
import { cpp } from '@codemirror/lang-cpp'
import { EditorView } from 'codemirror';

interface Props {
    height?: number;
}

const props = withDefaults(defineProps<Props>(), {
    height: 300
});


const fixedHeightEditor = EditorView.theme({
  "&": {height: `${props.height}px`},
  ".cm-scroller": {overflow: "auto"}
})


const extensions = [oneDark, cpp(), fixedHeightEditor]
const model = defineModel<string>();

</script>