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
  min-height: 80vh

.editor-panel
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
</style>

<script lang="ts">
import Vue from 'vue'
import { codemirror } from 'vue-codemirror'
import { EditorConfiguration } from 'codemirror'

import 'codemirror/lib/codemirror.css'
import 'codemirror/mode/python/python.js'
import 'codemirror/theme/dracula.css'
import 'codemirror/keymap/vim.js'

const cmOptions: EditorConfiguration = {
  tabSize: 2,
  mode: 'python',
  theme: 'dracula',
  lineNumbers: false,
  keyMap: 'vim',
  showCursorWhenSelecting: true,
  viewportMargin: Infinity,
}

const starterCode = `
def on_voice(text):
  if "sleep" in text:
    youtube("Sleeping and Meditation Music")
    say("Sleep tight!")
    return

  if "monday" in text:
    show_color("yellow")
    show_text("Good Morning! 🍹")
`.trim()

export default Vue.extend({
  components: { codemirror },

  data: () => ({
    code: starterCode,
    cmOptions,
  }),
})
</script>
