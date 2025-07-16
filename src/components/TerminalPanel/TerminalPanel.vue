<script setup>
import { FitAddon } from '@xterm/addon-fit'
import { Terminal } from '@xterm/xterm'
import { nextTick, onBeforeUnmount, onMounted, ref } from 'vue'
import darkThemeJson from './theme/xterm/vscode/DarkModern.json'
import '@xterm/xterm/css/xterm.css'

const props = defineProps({
  terminalDetail: Object,
  type: String,
})

const terminal = ref(null)
const fitAddon = new FitAddon()
let term = null

function fitTerm() {
  console.log('fitTerm')
  fitAddon.fit()
}

function initTerminal() {
  term = new Terminal({
    fontFamily: 'Consolas, \'Courier New\', monospace',
    cursorBlink: true,
    cursorStyle: 'block',
    theme: { ...darkThemeJson },
  })
  term.loadAddon(fitAddon)
  term.open(terminal.value)
  term.write('Hello from \x1B[1;3;31mxterm.js\x1B[0m $ ')

  term.onData((data) => {
    // if (socket && socket.readyState === WebSocket.OPEN) {
    //   socket.send(data)
    // }
    term.write(data)
  })
  nextTick(() => {
    fitTerm()
  })
}

function onTerminalResize() {
  window.addEventListener('resize', fitTerm)
}
function removeResizeListener() {
  window.removeEventListener('resize', fitTerm)
}

onMounted(() => {
  initTerminal()
  // initWebSocket()
  onTerminalResize()
})

onBeforeUnmount(() => {
  removeResizeListener()
})

defineExpose({
  fit() {
    nextTick(() => {
      fitTerm()
    })
  },
})
</script>

<template>
  <div class="h-full w-full relative p-0 overflow-hidden">
    <div ref="terminal" class="h-full w-full" />
  </div>
</template>

<style scoped>

</style>
