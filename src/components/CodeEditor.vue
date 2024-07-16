<template>
    <Codemirror class="code-editor" @ready="onReady" :extensions="extensions" v-model="model" />
</template>

<script setup lang="ts">
import { oneDark } from '@codemirror/theme-one-dark';
import { Codemirror } from 'vue-codemirror';
import { cpp } from '@codemirror/lang-cpp'
import { EditorView } from 'codemirror';
import {EditorState} from '@codemirror/state'

interface Props {
    height?: number;
    lineToSelect?: number;
}

const props = withDefaults(defineProps<Props>(), {
    height: 500,
    lineToSelect: undefined
});


const fixedHeightEditor = EditorView.theme({
  "&": {height: `${props.height}px`},
  ".cm-scroller": {overflow: "auto"}
})


const extensions = [oneDark, cpp(), fixedHeightEditor]
const model = defineModel<string>();


function selectLine(view: EditorView, state: EditorState, lineNumber: number){
  const line = state.doc.line(lineNumber);
  view.dispatch({
    selection: { head: line.from, anchor: line.to },
    scrollIntoView: true
  });
}

function onReady(payload: { view: EditorView; state: EditorState; container: HTMLDivElement}){
    if(props.lineToSelect){
        selectLine(payload.view, payload.state, props.lineToSelect);
    }
}

</script>