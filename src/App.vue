<script setup lang="ts">
import { Ref, ref } from 'vue';
import { marked } from "marked"
import MarkdownLine from './components/MarkdownLine.vue';
// import { Cursor } from './cursorHelpers'

const editor: Ref<null | HTMLDivElement> = ref(null)
const cursorPos = ref(0)
const lines = ref(marked.lexer(
  `# heading
This \`document\` is for _demonstartion_ of the editor only.

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


  return result.join("\n")
}

const handleChange = (e: KeyboardEvent) => {
  // Cursor.setCurrentCursorPosition(cursorPos.value + 1, editor.value);
  // editor.value?.focus();

  // cursorPos.value = Cursor.getCurrentCursorPosition(editor.value);



  const x = getContent()
  lines.value = marked.lexer(x, { gfm: true, breaks: true })

  let newTokens: marked.Token[] = []
  for (let x of lines.value) {
    newTokens = newTokens.concat(splitToken(x))
  }

  // @ts-ignore
  lines.value = newTokens

  // delete lines inserted by the browser
  for (let line of Array.from(document.getElementsByClassName("line"))) {
    let firstChildSaved = false
    for (let child of Array.from(line.children)) {
      if (firstChildSaved && child.textContent!.trim().length > 0) {
        console.log("d", child.textContent?.trim())

        line.removeChild(child)
      }
      firstChildSaved = true
    }
  }
  // setTimeout(() => {

  // }, 3);
}


</script>

<template>
  <div class="bg-gray-800 flex justify-center outline-none ">
    <main id="editor" ref="editor" class=" text-white h-screen p-8 max-w-[900px] w-full outline-none "
      @keyup="handleChange" contenteditable>
      <div>
        <div v-for="line in lines" class="line ">
          <h1 class="font-bold text-2xl w-full underline py-4" v-if="line.type === 'heading' && line.depth === 1">
            {{line.raw}}
          </h1>
          <h2 class="font-bold text-xl underline py-3" v-else-if="line.type === 'heading' && line.depth === 2">
            {{line.raw}}
          </h2>
          <h2 class="font-bold text-lg underline py-2" v-else-if="line.type === 'heading' && line.depth === 3">
            {{line.raw}}
          </h2>
          <blockquote class="pl-2 border-l-2" v-else-if="line.type === 'blockquote'">{{line.raw}}</blockquote>
          <p class="realline" v-else>
            <MarkdownLine :line="line" />
          </p>
        </div>
      </div>
    </main>
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
