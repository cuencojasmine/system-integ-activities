<template>
  <div class="scanner-bg d-flex justify-center align-start" style="min-height: 100vh; padding: 48px 16px">
    <v-container style="max-width: 480px;" class="pa-0">

      <!-- Webcam Card -->
      <v-card class="scanner-card mb-4" elevation="0" rounded="xl">

        <div class="card-header d-flex flex-column align-center justify-center">
          <v-icon size="44" color="white" class="mb-3">mdi-qrcode-scan</v-icon>
          <h1 class="header-title">QR Code Scanner</h1>
          <p class="header-sub">Scan via webcam or upload an image</p>
        </div>

        <v-card-text class="pa-6">

          <!-- Video Feed -->
          <div class="video-wrapper mb-4">
            <video ref="videoElem" playsinline muted class="qr-video" />
            <div v-if="isScanning" class="scan-overlay">
              <div class="scan-line" />
            </div>
          </div>

          <!-- Controls -->
          <div class="d-flex gap-3 mb-4">
            <v-btn
              v-if="!isScanning"
              rounded="xl"
              size="large"
              class="action-btn-primary"
              prepend-icon="mdi-play"
              block
              @click="startScan"
            >
              Start Scanning
            </v-btn>
            <v-btn
              v-else
              rounded="xl"
              size="large"
              class="action-btn-stop"
              prepend-icon="mdi-stop"
              block
              @click="stopScan"
            >
              Stop Scanning
            </v-btn>
          </div>

          <!-- Status chips -->
          <div class="d-flex align-center gap-2 mb-4 flex-wrap">
            <v-progress-circular
              v-if="isScanning"
              indeterminate
              color="#2B5748"
              size="16"
              width="2"
            />
            <v-chip
              size="small"
              variant="tonal"
              :style="isScanning ? 'background:#e6f2ee;color:#2B5748;' : 'background:#F0FDF8;color:#0F2D40;'"
            >
              {{ isScanning ? 'Scanning...' : 'Idle' }}
            </v-chip>
            <v-chip size="small" variant="tonal" style="background:#F0FDF8;color:#0F2D40;">
              Camera: {{ hasCamera ? '✅ Found' : '❌ None' }}
            </v-chip>
          </div>

          <!-- Webcam Result -->
          <div v-if="webcamResult" class="result-alert result-success">
            <v-icon size="20" style="color:#2B5748" class="mr-2">mdi-qrcode-edit</v-icon>
            <div>
              <div class="result-label">Detected QR Code</div>
              <div class="result-value">{{ webcamResult }}</div>
              <div class="result-time">Last detected at: {{ lastDetectedAt }}</div>
            </div>
          </div>

          <div v-else-if="isScanning" class="result-alert result-info">
            <v-icon size="20" style="color:#0F2D40" class="mr-2">mdi-information-outline</v-icon>
            <div class="result-value">Point your camera at a QR code...</div>
          </div>

        </v-card-text>
      </v-card>

      <!-- File Scan Card -->
      <v-card class="scanner-card" elevation="0" rounded="xl">

        <v-card-text class="pa-6">
          <p class="section-label mb-4">
            <v-icon size="18" class="mr-1" style="color:#2B5748">mdi-image-outline</v-icon>
            Scan from File
          </p>

          <v-file-input
            label="Choose an image"
            accept="image/*"
            prepend-icon=""
            prepend-inner-icon="mdi-upload-outline"
            variant="solo-filled"
            flat
            rounded="lg"
            density="comfortable"
            hide-details
            bg-color="#F0FDF8"
            class="mb-4"
            @change="scanFile"
          />

          <div
            v-if="fileResult"
            :class="['result-alert', fileResult === 'No QR code found.' ? 'result-warn' : 'result-success']"
          >
            <v-icon size="20" class="mr-2" :style="fileResult === 'No QR code found.' ? 'color:#b45309' : 'color:#2B5748'">
              mdi-qrcode
            </v-icon>
            <div>
              <div class="result-label">Result</div>
              <div class="result-value">{{ fileResult }}</div>
            </div>
          </div>

        </v-card-text>
      </v-card>

    </v-container>
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
const isScanning = ref(false)

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
      {
        returnDetailedScanResult: true,
        preferredCamera: 'environment',
        highlightScanRegion: true,
        highlightCodeOutline: true,
      }
    )
  }
})

const startScan = async () => {
  await qrScanner?.start()
  isScanning.value = true
  webcamResult.value = ''
}

const stopScan = () => {
  qrScanner?.stop()
  isScanning.value = false
}

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
.scanner-bg {
  background: #F0FDF8;
}

.scanner-card {
  background: #ffffff;
  border: 1px solid #c8e6d8;
  overflow: hidden;
}

.card-header {
  height: 190px;
  background: linear-gradient(135deg, #0F2D40 0%, #2B5748 100%);
  padding: 0 32px;
}

.header-title {
  font-size: 1.5rem;
  font-weight: 700;
  color: #ffffff;
  margin: 0;
  letter-spacing: -0.01em;
}

.header-sub {
  font-size: 0.85rem;
  color: rgba(255, 255, 255, 0.72);
  margin: 4px 0 0;
}

.section-label {
  font-size: 0.8rem;
  font-weight: 700;
  color: #0F2D40;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  display: flex;
  align-items: center;
}

/* Video */
.video-wrapper {
  position: relative;
  width: 100%;
  aspect-ratio: 1 / 1;
  background: #0F2D40;
  border-radius: 12px;
  overflow: hidden;
  border: 1px solid #c8e6d8;
}

.qr-video {
  width: 100%;
  height: 100%;
  object-fit: cover;
  display: block;
}

.scan-overlay {
  position: absolute;
  inset: 0;
  pointer-events: none;
}

.scan-line {
  position: absolute;
  left: 0;
  right: 0;
  height: 2px;
  background: rgba(43, 87, 72, 0.9);
  animation: scanMove 2s linear infinite;
  box-shadow: 0 0 10px rgba(43, 87, 72, 0.8);
}

@keyframes scanMove {
  0%   { top: 5%; }
  50%  { top: 95%; }
  100% { top: 5%; }
}

/* Buttons */
.action-btn-primary {
  background: linear-gradient(135deg, #0F2D40, #2B5748) !important;
  color: #ffffff !important;
  font-weight: 600;
}

.action-btn-stop {
  background-color: #fff1f0 !important;
  color: #c0392b !important;
  border: 1px solid #fecaca !important;
  font-weight: 600;
}

/* Result boxes */
.result-alert {
  display: flex;
  align-items: flex-start;
  padding: 14px 16px;
  border-radius: 12px;
}

.result-success {
  background: #e6f2ee;
  border: 1px solid #c8e6d8;
}

.result-info {
  background: #F0FDF8;
  border: 1px solid #c8e6d8;
}

.result-warn {
  background: #fffbeb;
  border: 1px solid #fde68a;
}

.result-label {
  font-size: 0.75rem;
  color: #6b7280;
  font-weight: 500;
  margin-bottom: 2px;
}

.result-value {
  font-size: 0.875rem;
  font-weight: 600;
  color: #0F2D40;
  word-break: break-all;
}

.result-time {
  font-size: 0.75rem;
  color: #6b7280;
  margin-top: 4px;
}
</style>