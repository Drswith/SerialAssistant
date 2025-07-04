<script setup>
import { refThrottled, useScroll } from '@vueuse/core'
import dayjs from 'dayjs'
import { computed, ref, watch } from 'vue'
import { useDataCode } from '@/composables/useDataCode/useDataCode'
import { useRecordStore } from '@/store/useRecordStore'
import { useSerialStore } from '@/store/useSerialStore'

const { bufferToDecFormat, bufferToHexFormat, bufferToString, stringToHtml } = useDataCode()
const { records, readingRecord, pinBottom } = useRecordStore()
const { recordTypes } = useSerialStore()
const rootEl = ref(null)
const { x, y, isScrolling, arrivedState, directions } = useScroll(rootEl, {
  behavior: 'smooth',
})
async function scrollToBottom() {
  rootEl.value.scrollTop = rootEl.value.scrollHeight + 2000
}

async function toggoleRecordDisplay(record) {
  const index = recordTypes.value.indexOf(record.display)
  record.display = recordTypes.value[(index + 1) % recordTypes.value.length]
}
const recordLength = computed(() => records.value.length)
watch(
  [recordLength, refThrottled(readingRecord, 150)],
  (value, oldValue) => {
    if (pinBottom.value & (value[0] != oldValue[0])) {
      setTimeout(() => scrollToBottom(), 0)
    }
  },
  { deep: true },
)
</script>

<template>
  <div ref="rootEl" class="overflow-y-auto scroll-smooth p-2 relative record-panel pb-10">
    <template v-for="record in records" :key="record.time">
      <div v-if="record.type == 'read'" class="chat chat-start">
        <div class="chat-header mx-2 flex">
          <!-- TODO 点击可以切换时间显示格式(是否显示日期) -->
          <div class="text-sm opacity-70">
            {{ dayjs(record.time).format("HH:mm:ss:SSS") }}
          </div>
          <div class="w-4" />
          <div class="cursor-pointer" @click="() => toggoleRecordDisplay(record)">
            {{ record.display }}
          </div>
        </div>
        <div class="chat-bubble break-words text-sm">
          <div v-if="record.display == 'hex'">
            {{ bufferToHexFormat(record.data) }}
          </div>
          <div v-else-if="record.display == 'ascii'" v-html="stringToHtml(bufferToString(record.data))" />
          <div v-else>
            {{ bufferToDecFormat(record.data) }}
          </div>

          <!-- {{
            record.display == "hex"
              ? bufferToHexFormat(record.data)
              : stringToHtml(bufferToStr(record.data))
          }} -->
        </div>
      </div>
      <div v-if="record.type == 'write'" class="chat chat-end">
        <div class="chat-header mx-2 flex">
          <!-- TODO 点击可以切换时间显示格式(是否显示日期) -->
          <div class="text-sm opacity-70">
            {{ dayjs(record.time).format("HH:mm:ss:SSS") }}
          </div>
          <div class="w-4" />
          <div class="cursor-pointer" @click="() => toggoleRecordDisplay(record)">
            {{ record.display }}
          </div>
        </div>
        <div class="chat-bubble break-words text-sm">
          <div v-if="record.display == 'hex'">
            {{ bufferToHexFormat(record.data) }}
          </div>
          <div v-else-if="record.display == 'ascii'" v-html="stringToHtml(bufferToString(record.data))" />
          <div v-else>
            {{ bufferToDecFormat(record.data) }}
          </div>
        </div>
      </div>
    </template>
    <div v-if="readingRecord" class="chat chat-start">
      <div class="chat-header mx-2">
        <!-- TODO 点击可以切换时间显示格式(是否显示日期) -->
        <div class="text-sm opacity-70">
          {{ dayjs(readingRecord.time).format("HH:mm:ss:SSS") }}
        </div>
      </div>
      <div class="chat-bubble break-all text-sm">
        <div v-if="readingRecord.display == 'hex'">
          {{ bufferToHexFormat(readingRecord.data) }}
        </div>
        <div v-else v-html="stringToHtml(bufferToString(readingRecord.data))" />
      </div>
    </div>
  </div>
</template>

<style scoped>
/* scroolbar like apple */
::-webkit-scrollbar {
  width: 4px;
  height: 8px;
}

::-webkit-scrollbar-thumb {
  background: #ccc;
  border-radius: 4px;
}

::-webkit-scrollbar-track {
  border-radius: 4px;
  margin: 15px;
}

::-webkit-scrollbar-button {
  width: 0;
  height: 0;
}
</style>
