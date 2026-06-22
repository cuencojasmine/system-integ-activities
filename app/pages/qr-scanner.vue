<template>
  <div class="scanner-container">
    <h1>QR Code Scanner</h1>

    <h2>Scan from WebCam:</h2>
    <video ref="videoElem" style="width: 100%; max-width: 400px;"></video>

    <div class="controls">
      <button @click="startScan">Start Scanning</button>
      <button @click="stopScan">Stop Scanning</button>
    </div>

    <p><b>Device has camera:</b> {{ hasCamera }}</p>
    <p><b>Detected QR code:</b> {{ webcamResult || 'None' }}</p>
    <p><b>Last detected at:</b> {{ lastDetectedAt || '—' }}</p>

    <hr />

    <h2>Scan from File:</h2>
    <input type="file" accept="image/*" @change="scanFile" />
    <p><b>Detected QR code:</b> {{ fileResult || 'None' }}</p>

  </div>
</template>

<script setup lang="ts">

import { ref, onMounted, onUnmounted } from 'vue'
import QrScanner from 'qr-scanner'

const videoElem = ref<HTMLVideoElement | null>(null)
const webcamResult = ref('')
const fileResult = ref('')
const lastDetectedAt = ref('')
const hasCamera = ref(false)

let qrScanner: QrScanner | null = null

onMounted(async () => {
  hasCamera.value = await QrScanner.hasCamera()

  if (videoElem.value) {
    qrScanner = new QrScanner(
      videoElem.value,
      (res) => {
        webcamResult.value = res.data
        lastDetectedAt.value = new Date().toLocaleTimeString()
      },
      { returnDetailedScanResult: true }
    )
  }
})

const startScan = () => qrScanner?.start()
const stopScan = () => qrScanner?.stop()

const scanFile = async (event: Event) => {
  const file = (event.target as HTMLInputElement).files?.[0]
  if (!file) return
  try {
    const result = await QrScanner.scanImage(file, { returnDetailedScanResult: true })
    fileResult.value = result.data
  } catch {
    fileResult.value = 'No QR code found.'
  }
}

onUnmounted(() => {
  qrScanner?.destroy()
  qrScanner = null
})
</script>

<style scoped>
.scanner-container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  padding: 20px;
  max-width: 500px;
  margin: 0 auto;
}

h1 {
  margin-bottom: 8px;
}

h2 {
  margin-top: 16px;
  margin-bottom: 8px;
}

.controls {
  margin-top: 12px;
  display: flex;
  gap: 10px;
}

button {
  padding: 8px 16px;
  cursor: pointer;
}

hr {
  width: 100%;
  margin: 24px 0;
  border: none;
  border-top: 1px solid #ccc;
}

input[type="file"] {
  margin-top: 4px;
}

p {
  margin: 6px 0;
}
</style>