<script setup lang="ts">
import { ref } from 'vue'
import { marked } from "marked"


defineProps<{ line: marked.Token }>()

const count = ref(0)
</script>

<template>
  <span v-if="'tokens' in line" v-for="token in line.tokens">
    <b v-if="token.type === 'strong'">{{ token.raw }}</b>
    <strong contenteditable="false" v-else-if="token.type === 'br'">{{ token.raw }}</strong>
    <em class="font-italic" v-else-if="token.type === 'em'">{{ token.raw }}</em>
    <pre class="bg-gray-600 inline p-1 rounded-sm" v-else-if="token.type === 'codespan'">{{ token.raw }}</pre>


    <span v-else>{{ token.raw }}</span>
  </span>
</template>

