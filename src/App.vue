<template>
  <div class="app-shell">
    <header class="topbar">
      <p class="eyebrow">Ticket Scanner</p>
      <h1>التحقق من التذكرة</h1>
      <p class="subtitle">وجه الكاميرا نحو رمز QR للتحقق من صلاحية التذكرة</p>
    </header>

    <div class="scanner-card">
      <qrcode-stream :camera="camera" @decode="onDecode" @init="onInit" class="scanner">
        <div v-show="cameraForzen" class="validation-layer">
          <div class="validation-notice" v-if="validating">
            جاري التحقق من التذكرة...
          </div>

          <div class="validation-result" v-if="!validating">
            <div v-if="isValid" class="valid">
              التذكرة صالحة
            </div>

            <div v-else class="invalid">
              التذكرة غير صالحة
            </div>
          </div>
        </div>
      </qrcode-stream>

      <div v-if="!loading && !camera" class="fallback">
        <p>جاهز لفتح الكاميرا</p>
      </div>
    </div>

    <div class="result-box">
      <strong>آخر نتيجة:</strong>
      <span>{{ result || 'لم يتم مسح أي رمز بعد' }}</span>
    </div>
  </div>
</template>

<script>
import { firestore } from "./firebase";
export default {
  data () {
    return {
      isValid: false,
      validating: false,
      camera: null,
      result: null,
      loading: false,
      firstLoad: true
    }
  },

  computed: {
    cameraForzen () {
      return this.camera === false || (this.loading && !this.firstLoad)
    }
  },

  methods: {
    async onDecode (content) {
      this.result = content
      this.stopCamera()

      this.validating = true
      this.isValid = await this.validate(content)
      this.validating = false

      window.setTimeout(() => {
        this.startCamera()
      }, 2000)
    },

    stopCamera () {
      this.camera = false
    },

    startCamera () {
      this.camera = null
    },

    validate (content) {
      return firestore
        .collection("tickets")
        .where("ticketid", "==", content)
        .get()
        .then(snapshot => {
          return new Promise(resolve => {
            if (!snapshot.empty) {
              return resolve(true)
            }
            return resolve(false)
          })
        })
    },

    async onInit (promise) {
      this.loading = true

      try {
        await promise
      } catch (error) {
        // eslint-disable-next-line
        console.error(error)
      } finally {
        this.firstLoad = false
        this.loading = false
      }
    }
  }
}
</script>

<style scoped>
.app-shell {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #f3f7ff 0%, #eaf4ff 100%);
  padding: 24px;
  font-family: Arial, sans-serif;
  color: #123;
}

.topbar {
  text-align: center;
  margin-bottom: 20px;
}

.eyebrow {
  margin: 0;
  color: #1d4ed8;
  font-size: 0.8rem;
  font-weight: bold;
  letter-spacing: 1px;
  text-transform: uppercase;
}

h1 {
  margin: 10px 0 8px;
  font-size: clamp(2rem, 4vw, 3rem);
}

.subtitle {
  margin: 0;
  color: #4b5563;
  font-size: 1rem;
}

.scanner-card {
  position: relative;
  width: min(90vw, 520px);
  min-height: 420px;
  border-radius: 18px;
  overflow: hidden;
  background: #0f172a;
  box-shadow: 0 18px 45px rgba(15, 23, 42, 0.2);
  border: 2px solid rgba(255,255,255,0.15);
}

.scanner {
  display: block;
  width: 100%;
  min-height: 420px;
}

.fallback {
  position: absolute;
  inset: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  background: rgba(15, 23, 42, 0.75);
  color: #fff;
  font-size: 1.1rem;
  text-align: center;
  padding: 20px;
}

.validation-layer {
  position: absolute;
  inset: 0;
  background-color: rgba(255, 255, 255, 0.85);
  text-align: center;
  padding: 10px;
  display: flex;
  flex-flow: column nowrap;
  justify-content: center;
}

.validation-notice,
.validation-result {
  font-weight: bold;
  font-size: 1.4rem;
}

.valid {
  color: green;
}

.invalid {
  color: red;
}

.result-box {
  margin-top: 18px;
  width: min(90vw, 520px);
  background: rgba(255, 255, 255, 0.82);
  border-radius: 12px;
  padding: 14px 16px;
  box-shadow: 0 8px 18px rgba(0, 0, 0, 0.08);
  color: #1f2937;
  word-break: break-all;
}

.result-box strong {
  margin-left: 8px;
}
</style>