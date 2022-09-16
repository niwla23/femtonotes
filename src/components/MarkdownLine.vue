<script setup lang="ts">
import { ref } from 'vue'
import { marked } from "marked"


defineProps<{ line: marked.Token }>()

const count = ref(0)
</script>

<template>
  <span v-if="'tokens' in line" v-for="token in line.tokens">
    <b v-if="token.type === 'strong'">{{ token.raw }}</b>
    <b v-else-if="token.type === 'br'">{{ token.raw }}<br /></b>
    <span v-else>{{ token.raw }}</span>
  </span>
</template>

