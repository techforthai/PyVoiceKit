<template lang="pug">
  .app
    .panel.display-panel
      h1 Voice Kit
    .panel.editor-panel
      .editor-titlebar
        .editor-window-decoration
          .dot.red
          .dot.yellow
          .dot.green
        .editor-title Code Editor
      .editor-container
        codemirror.editor-area(v-model="code" :options="cmOptions")
        button.btn-run-code(@click="runCode") Run Code
      .console-container
        .log-container
          .log(v-for="log in logs" :class="log.type") {{log.text}}
</template>

<style lang="sass" scoped>
.app
  display: flex
  width: 100%

  min-height: 100vh
  font-family: "JetBrains Mono", monospace

.display-panel
  display: flex
  align-items: center
  justify-content: center
  min-height: 100vh
  background: #9b59b6
  width: 60%

  h1
    color: white
    font-size: 40px
    margin: 0

.editor-area
  height: 600px

.editor-panel
  position: relative
  background: #282a36
  box-shadow: 1px 1px 8px 4px rgba(0, 0, 0, 0.3)
  z-index: 1
  width: 40%
  margin: 25px
  border-radius: 6px

  .editor-titlebar
    display: flex
    justify-content: center
    position: relative
    text-align: center
    background: #21232c
    padding: 8px 0px
    margin-bottom: 10px
    box-shadow: 0px 0px 5px rgba(0, 0, 0, 0.3)
    z-index: 2
    border-radius: 6px 6px 0px 0px

  .editor-title
    color: white
    font-size: 16px
    text-align: center

  .editor-container
    padding: 5px 20px
    height: 100%

  .editor-window-decoration
    position: absolute
    left: 10px
    top: 11px
    display: flex
    align-items: center
    justify-content: center

.dot
  width: 14px
  height: 14px
  border-radius: 50%
  box-shadow: 1px 1px 5px rgba(0, 0, 0, 0.3)
  margin-left: 10px

.dot.red
  background: #fa5b52

.dot.yellow
  background: #e6c02b

.dot.green
  background: #53c32b

button.btn-run-code
  font-size: 24px
  font-family: "JetBrains Mono", monospace
  color: white
  background: linear-gradient(-225deg, rgb(241, 95, 121) 35%, rgb(178, 69, 146) 100%)
  cursor: pointer
  border: none
  padding: 8px 15px
  position: absolute
  bottom: 18px
  right: 20px
  box-shadow: 1px 1px 6px rgba(0, 0, 0, 0.5)
  border-radius: 8px
  z-index: 10
  transition: 1s all cubic-bezier(0.19, 0.16, 0, 1.27)
  outline: none

  &:hover
    transform: scale(1.15)

.console-container
  position: absolute
  bottom: 0
  width: 100%
  height: 150px
  overflow: scroll
  background: #21232c
  box-shadow: 0px -6px 8px rgba(0, 0, 0, 0.1)

.log-container
  padding: 20px
  font-size: 20px
  line-height: 1.5

.log.info
  color: white

.log.error
  color: #fa5b52
</style>

<script lang="ts">
import Vue from 'vue'
import { codemirror } from 'vue-codemirror'
import { EditorConfiguration } from 'codemirror'

import { app } from '../main'

import 'codemirror/lib/codemirror.css'
import 'codemirror/mode/python/python.js'
import 'codemirror/theme/dracula.css'
import 'codemirror/keymap/vim.js'

const cmOptions: EditorConfiguration = {
  tabSize: 2,
  mode: 'python',
  theme: 'dracula',
  lineNumbers: false,
  // keyMap: 'vim',
  showCursorWhenSelecting: false,
  viewportMargin: Infinity,
}

const starterCode = `print("Hello, World!")`.trim()

function run(code: string) {
  if (!window.__BRYTHON__) return

  const js = window.__BRYTHON__.python_to_js(code)

  console.groupCollapsed('ℹ️ Translating Python to JavaScript')
  console.log('Python:', code)
  console.log('JavaScript:', js)
  console.groupEnd()

  return eval.call(this, js)
}

window.run = run

export default Vue.extend({
  components: { codemirror },

  data: () => ({
    code: starterCode,
    cmOptions,
    logs: [],
  }),

  mounted() {
    window.app = this
  },

  methods: {
    patch() {
      window.__BRYTHON__.builtins.print = (...args) => {
        console.log(`[Python] ${args.join('\n')}`)
        this.logs = [...this.logs, { text: args.join('\n'), type: 'info' }]
      }
    },
    runCode() {
      try {
        this.patch()
        run(this.code)
      } catch (error) {
        console.warn('User Code Error!', error.message)

        if (error.$py_error) {
          window.error = error
          this.logs = [
            ...this.logs,
            { text: `${error.msg} at line ${error.lineno}`, type: 'error' },
          ]
        }
      }
    },
  },
})
</script>
