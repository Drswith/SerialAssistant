<script setup>
import { debounce } from 'lodash'
import { onBeforeUnmount, onMounted, ref, watch } from 'vue'
import { Terminal } from 'xterm'
import { FitAddon } from 'xterm-addon-fit'
import 'xterm/css/xterm.css'

const props = defineProps({
  terminalDetail: Object,
  type: String,
})
const terminal = ref(null)
const fitAddon = new FitAddon()

const first = ref(true)
const loading = ref(true)
const terminalSocket = ref(null)
const term = ref(null)

function runRealTerminal() {
  loading.value = false
}

function onWSReceive(message) {
  // 首次接收消息,发送给后端，进行同步适配
  if (first.value === true) {
    first.value = false
    resizeRemoteTerminal()
  }
  const data = JSON.parse(message.data)
  term.value.element && term.value.focus()
  term.value.write(data.Data)
}

function errorRealTerminal(ex) {
  let message = ex.message
  if (!message)
    message = 'disconnected'
  term.value.write(`\x1B[31m${message}\x1B[m\r\n`)
  console.log('err')
}
function closeRealTerminal() {
  console.log('close')
}

function createWS() {
  const url = 'XXXX'
  terminalSocket.value = new WebSocket(url)
  terminalSocket.value.onopen = runRealTerminal
  terminalSocket.value.onmessage = onWSReceive
  terminalSocket.value.onclose = closeRealTerminal
  terminalSocket.value.onerror = errorRealTerminal
}
function initWS() {
  if (!terminalSocket.value) {
    createWS()
  }
  if (terminalSocket.value && terminalSocket.value.readyState > 1) {
    terminalSocket.value.close()
    createWS()
  }
}
// 发送给后端,调整后端终端大小,和前端保持一致,不然前端只是范围变大了,命令还是会换行
function resizeRemoteTerminal() {
  const { cols, rows } = term.value
  if (isWsOpen()) {
    terminalSocket.value.send(JSON.stringify({
      Op: 'resize',
      Cols: cols,
      Rows: rows,
    }))
  }
}
function initTerm() {
  term.value = new Terminal({
    lineHeight: 1.2,
    fontSize: 12,
    fontFamily: 'Monaco, Menlo, Consolas, \'Courier New\', monospace',
    theme: {
      background: '#181d28',
    },
    // 光标闪烁
    cursorBlink: true,
    cursorStyle: 'underline',
    scrollback: 100,
    tabStopWidth: 4,
  })
  term.value.open(terminal.value)
  term.value.loadAddon(fitAddon)
  // 不能初始化的时候fit,需要等terminal准备就绪,可以设置延时操作
  setTimeout(() => {
    fitAddon.fit()
  }, 5)
}
// 是否连接中0 1 2 3
function isWsOpen() {
  const readyState = terminalSocket.value && terminalSocket.value.readyState
  return readyState === 1
}
function fitTerm() {
  fitAddon.fit()
}
const onResize = debounce(() => fitTerm(), 800)

function termData() {
  // 输入与粘贴的情况,onData不能重复绑定,不然会发送多次
  term.value.onData((data) => {
    if (isWsOpen()) {
      terminalSocket.value.send(
        JSON.stringify({
          Op: 'stdin',
          Data: data,
        }),
      )
    }
  })
}
function onTerminalResize() {
  window.addEventListener('resize', onResize)
}
function removeResizeListener() {
  window.removeEventListener('resize', onResize)
}
// 监听类型变化，重置term
watch(() => props.type, () => {
  first.value = true
  loading.value = true
  terminalSocket.value = null
  initWS()
  // 重置
  term.value.reset()
})

onMounted(() => {
  initWS()
  initTerm()
  termData()
  onTerminalResize()
})
onBeforeUnmount(() => {
  removeResizeListener()
  terminalSocket.value && terminalSocket.value.close()
})
</script>

<template>
  <div
    ref="terminal"
    v-loading="loading"
    element-loading-text="拼命连接中"
  />
</template>

<style lang="scss" scoped>
#terminal {
  width: 100%;
  height: 100%;
}
</style>
