<script setup lang="ts">
import { Ref, ref } from 'vue';
import { marked } from "marked"
import MarkdownLine from './components/MarkdownLine.vue';

const editor: Ref<null | HTMLDivElement> = ref(null)
const lines = ref(marked.lexer(
  `# heading
This document is for demonstartion of the editor only.

## Heading 2

Lets put some **fat** text here
`
))

// this splits a token when the subtoken 'br' is found.
const splitToken = (token: marked.Token) => {
  if (!("tokens" in token)) {
    return [token]
  }

  let tokens: marked.Token[] = []
  let currentChildTokens: marked.Token[] = []
  for (let childToken of token.tokens!) {
    if (childToken.type === 'br') {
      tokens.push({ ...token, tokens: currentChildTokens })
      currentChildTokens = []
    } else {
      currentChildTokens.push(childToken)
    }
  }
  tokens.push({ ...token, tokens: currentChildTokens })

  return tokens
}

const getContent = () => {
  let result: string[] = []
  if (!editor.value) {
    return ""
  }
  document.querySelectorAll(".line").forEach((block) => {
    for (const child of Array.from(block.children)) {
      result.push(child.textContent?.trimEnd()!)
    }
  })


  console.log(result)
  return result.join("\n")
}

const handleChange = () => {
  const x = getContent()
  console.log(x)
  lines.value = marked.lexer(x, { gfm: true, breaks: true })

  let newTokens: marked.Token[] = []
  for (let x of lines.value) {
    newTokens = newTokens.concat(splitToken(x))
  }

  console.log("newTokens", newTokens)
  lines.value = newTokens

  console.log(lines.value)
}

</script>

<template>
  <div id="editor" ref="editor" class="bg-gray-800 text-white h-screen p-16 w-full" @keyup="handleChange"
    contenteditable>
    <div v-for="line in lines" class="outline-none line ">
      <h1 class="font-bold text-2xl w-full underline py-4" v-if="line.type === 'heading' && line.depth === 1">
        {{line.raw}}</h1>
      <h2 class="font-bold text-xl underline py-3" v-else-if="line.type === 'heading' && line.depth === 2">{{line.raw}}
      </h2>
      <h2 class="font-bold text-lg underline py-2" v-else-if="line.type === 'heading' && line.depth === 3">{{line.raw}}
      </h2>
      <blockquote class="pl-2 border-l-2" v-else-if="line.type === 'blockquote'">{{line.raw}}</blockquote>
      <p v-else>
        <MarkdownLine :line="line" />
      </p>
      <!-- <br />
      <br /> -->

    </div>
  </div>
</template>

<style scoped>
.logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
}

.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}

.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
}
</style>
