<template lang="pug">
  .app.backdrop(:style="backdropStyle")
    .panel.display-panel(v-if="!isFullEditor" :class="displayPanelClass")
      h1 {{message}}
      input.text-field(v-model="textField" @keyup.enter="submitText" :placeholder="lastText")
      .div(v-if="youtubeId")
        iframe.youtube-frame(width="500" height="500" :src="youtubeSrc" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen)
      button.btn-start-speech(@click="startSpeech" :class="microphoneClass")
        i.far.fa-microphone
      button.btn-show-editor(@click="toggleEditor" v-if="!isShowingEditor") Show Editor
    .panel.editor-panel(v-if="isShowingEditor" :class="editorPanelClass")
      .editor-titlebar
        .editor-window-decoration
          .dot.red(@click="toggleEditor")
          .dot.yellow
          .dot.green(@click="toggleFullEditor")
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
  padding: 35px 0

  min-height: 100vh
  font-family: "JetBrains Mono", monospace

.display-panel
  display: flex
  flex-direction: column
  align-items: center
  justify-content: center
  min-height: 100vh
  width: 60%

  h1
    color: white
    font-size: 40px
    margin: 0

.editor-area
  height: 600px

.text-field
  margin-top: 30px
  font-size: 25px
  border: none
  border-radius: 50em
  padding: 10px 15px
  outline: none
  width: 70%
  color: #121
  padding-left: 25px
  font-family: 'JetBrains Mono', sans-serif

.backdrop
  background-size: cover
  background-repeat: no-repeat
  background-attachment: fixed
  background-position: center center

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
  cursor: pointer
  background: #fa5b52

.dot.yellow
  background: #e6c02b

.dot.green
  cursor: pointer
  background: #53c32b

.is-listening
  transform: scale(1.5)

button.btn-show-editor
  position: fixed
  top: 20px
  right: 20px
  z-index: 20
  padding: 10px 15px
  font-size: 1.3em
  background: linear-gradient(-225deg, #f15f79 35%, #b24592 100%)
  color: white
  border: none
  box-shadow: 0px 0px 4px rgba(0, 0, 0, 0.5)
  border-radius: 4px
  outline: none
  font-family: "JetBrains Mono", monospace
  cursor: pointer

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

button.btn-start-speech
  border: none
  font-size: 36px
  background: #e74c3c
  color: white
  width: 60px
  height: 60px
  border-radius: 50%
  margin-top: 40px
  cursor: pointer
  outline: none
  box-shadow: 1px 1px 10px rgba(0, 0, 0, 0.3)
  transition: 1s all cubic-bezier(0.33, 0.39, 0, 1.49)

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

.youtube-frame
  margin-top: 60px
  box-shadow: 0px 0px 8px rgba(0, 0, 0, 0.5)
</style>

<script lang="ts">
import Vue from 'vue'
import { codemirror } from 'vue-codemirror'
import { EditorConfiguration } from 'codemirror'
import { Howl, Howler } from 'howler'

import { app } from '../main'

import 'codemirror/lib/codemirror.css'
import 'codemirror/mode/python/python.js'
import 'codemirror/theme/dracula.css'
import 'codemirror/keymap/vim.js'

const SpeechRecognition = SpeechRecognition || webkitSpeechRecognition
const SpeechGrammarList = SpeechGrammarList || webkitSpeechGrammarList
const SpeechRecognitionEvent =
  SpeechRecognitionEvent || webkitSpeechRecognitionEvent

const speech = new SpeechRecognition()
window.speech = speech

speech.lang = 'th-TH'
// speech.continuous = true

const isThai = (text: string) =>
  Array.from(text).some(x => /[\u0E00-\u0E7F]/.test(x))

const cmOptions: EditorConfiguration = {
  tabSize: 2,
  mode: 'python',
  theme: 'dracula',
  lineNumbers: false,
  // keyMap: 'vim',
  showCursorWhenSelecting: false,
  viewportMargin: Infinity
}

const starterCode = `
print("Hello, World!")
setText("Welcome to Voice Kit!")
setColor("#8e44ad")
`.trim()

function withExtras(code: string) {
  const extras = `
from browser import window

def setColor(color):
  window.eval("setColor(\`%s\`)" % color)

def setImage(image):
  window.eval("setImage(\`%s\`)" % image)

def setText(text):
  window.eval("setText(\`%s\`)" % text)

def youtube(youtubeId):
  window.eval("youtube(\`%s\`)" % youtubeId)

def speak(text): 
  print("H")

def listen(): 
  return window.eval("listen()")

def say(text): 
  window.eval("say(\`%s\`)" % text)

def playSound(url):
  window.eval("playSound(\`%s\`)" % url)
`.trim()

  return extras + '\n' + code
}

function run(code: string) {
  if (!window.__BRYTHON__) return

  const js = window.__BRYTHON__.python_to_js(withExtras(code))

  console.groupCollapsed('ℹ️ Translating Python to JavaScript')
  console.log('Python:', code)
  console.log('JavaScript:', js)
  console.groupEnd()

  return eval.call(this, js)
}

function speak(text) {
  const sid = Math.random().toString(36)

  const u = new SpeechSynthesisUtterance(text)
  u.volume = 1 // 0 - 1
  u.pitch = 1 // 0 - 2
  u.rate = 1 // 0 - 10

  if (isThai(text)) {
    const Kanya = speechSynthesis.getVoices().find(x => x.voiceURI === 'Kanya')
    u.voice = Kanya
    u.lang = 'th-TH'
  }

  u.onerror = e => {
    console.error('TTS Error:', e)
  }

  u.onstart = e => speech.stop()

  speechSynthesis.speak(u)

  return u
}

window.run = run
window.speak = speak

const saveEditor = (code: string) => localStorage.setItem('editor.code', code)

export default Vue.extend({
  components: { codemirror },

  data: () => ({
    code: starterCode,
    cmOptions,
    logs: [],
    message: 'Voice Kit',
    color: '#9b59b6',
    youtubeId: '',
    transcripts: [],
    isShowingEditor: false,
    isFullEditor: false,
    textField: '',
    isListening: false
  }),

  mounted() {
    const savedCode = localStorage.getItem('editor.code')
    if (savedCode) this.code = savedCode

    window.setText = (text: string) => (this.message = text)
    window.setColor = (color: string) => (this.color = color)
    window.setImage = (image: string) => (this.color = `url(${image})`)

    window.youtube = (query: string) => {
      const match = query.match(/youtube\.com\/watch\?v\=(\w+)/)
      if (!match) return

      this.youtubeId = match[1]
    }

    window.playSound = (url: string) => {
      const sound = new Howl({ src: [url] })

      sound.play()
    }

    window.listen = (text: string) =>
      this.transcripts[this.transcripts.length - 1]

    window.say = (text: string) => speak(text)

    speech.onstart = () => {
      console.log('👂🏻 listening...')
      this.isListening = true
    }

    speech.onresult = e => {
      const text = e.results[0][0].transcript
      console.log('💬:', text)

      this.transcripts = [...this.transcripts, text]
      this.runCode()

      console.log('Transcripts:', this.transcripts)
      this.isListening = false
    }

    setInterval(() => saveEditor(this.code), 2000)
  },

  methods: {
    startSpeech() {
      if (this.isListening) return

      speech.start()
    },
    toggleEditor() {
      this.isShowingEditor = !this.isShowingEditor
    },
    toggleFullEditor() {
      this.isFullEditor = !this.isFullEditor
    },
    submitText() {
      this.transcripts = [...this.transcripts, this.textField]
      this.runCode()
    },
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
        window.error = error
        console.warn('User Code Error!', error.message)

        if (error.$py_error) {
          this.logs = [
            ...this.logs,
            {
              text:
                `${error.args[0]}` +
                (error.lineno ? `at line ${error.lineno}` : ''),
              type: 'error'
            }
          ]
        }
      }
    }
  },

  computed: {
    backdropStyle() {
      return {
        background: this.color
      }
    },

    displayPanelClass() {
      return {
        'is-full-width': !this.isShowingEditor
      }
    },

    editorPanelClass() {
      return {
        'is-full-width': this.isFullEditor
      }
    },

    microphoneClass() {
      return {
        'is-listening': this.isListening
      }
    },

    youtubeSrc() {
      return `https://www.youtube.com/embed/${this.youtubeId}?autoplay=1&showinfo=0&controls=0`
    },

    lastText() {
      return this.transcripts[this.transcripts.length - 1]
    }
  }
})
</script>
