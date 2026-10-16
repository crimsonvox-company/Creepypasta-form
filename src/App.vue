<template>
  <div class="app" :class="wide ? 'app--wide' : 'app--col'">
    <!-- Latar belakang utama (tetap di tempat saat halaman di-scroll) -->
    <div
      class="app__bg"
      :style="{ backgroundImage: `url(${bgMain})`, backgroundPosition: wide ? 'center top' : 'center bottom' }"
      aria-hidden="true"
    ></div>

    <Transition name="fade" mode="out-in">
      <!-- ========================================== -->
      <!-- 1. LOADING SCREEN (SPLASH) -->
      <!-- ========================================== -->
      <section v-if="currentScreen === 'loading'" key="loading" class="loading" aria-live="polite">
        <div class="logo-badge logo-badge--xl animate-glow">
          <img :src="logoImg" alt="Logo" />
        </div>
        <div
          class="loading__bar"
          role="progressbar"
          aria-valuemin="0"
          aria-valuemax="100"
          :aria-valuenow="Math.round(progress)"
        >
          <div class="loading__fill" :style="{ width: progress + '%' }"></div>
        </div>
        <p class="loading__text">Memuat...</p>
      </section>

      <!-- ========================================== -->
      <!-- 2 & 3. FORM + SUKSES (satu bingkai yang sama) -->
      <!-- ========================================== -->
      <div v-else key="main" class="main">
        <div class="stage" :class="wide ? 'stage--wide' : 'stage--col'" :style="wide ? stageStyle : undefined">
          <!-- Lendir samping: cermin dari lendir bingkai, supaya menyambung di layar ultra lebar -->
          <template v-if="wide">
            <img :src="slimeSide" alt="" class="slime slime--left" aria-hidden="true" />
            <img :src="slimeSide" alt="" class="slime slime--right" aria-hidden="true" />
            <img :src="frameImg" alt="" class="frame" aria-hidden="true" />
          </template>

          <!-- Logo: pojok kiri atas, sama di desktop dan mobile -->
          <div class="logo-badge" :class="wide ? 'logo-badge--wide' : 'logo-badge--col'">
            <img :src="logoImg" alt="Logo" />
          </div>

          <!-- Judul + kartu. Di desktop pembungkus ini "hilang" (display: contents) -->
          <div class="group" :class="wide ? 'group--wide' : 'group--col'">
            <!-- Judul (hanya di layar portrait; di desktop sudah menyatu dengan bingkai) -->
            <img v-if="!wide" :src="titleImg" alt="Halloween - Bisikan Creepy Pasta" class="title" />

            <!-- Kartu: transparan di atas bingkai (desktop) atau papan CSS (mobile) -->
            <div class="card" :class="wide ? 'card--wide' : 'card--col'">
              <Transition name="swap" mode="out-in">
                <form
                  v-if="currentScreen === 'form'"
                  key="form"
                  class="story-form"
                  novalidate
                  @submit.prevent="submitStory"
                >
                  <div class="field-row">
                    <!-- Nama -->
                    <div class="field field--name">
                      <div class="label">
                        <label for="f-name">Nama Penulis / Samaran *</label>
                        <span v-if="errors.name" class="err" role="alert">{{ errors.name }}</span>
                      </div>
                      <input
                        id="f-name"
                        ref="nameInput"
                        v-model="form.name"
                        type="text"
                        autocomplete="nickname"
                        maxlength="60"
                        class="input"
                        :class="{ shake: shaking && errors.name }"
                        :aria-invalid="!!errors.name"
                        @focus="focusField = 'name'"
                        @blur="focusField = 'idle'"
                        @input="errors.name = ''"
                      />
                    </div>

                    <!-- Instagram -->
                    <div class="field">
                      <div class="label">
                        <label for="f-ig">Instagram (Opsional)</label>
                        <span v-if="errors.instagram" class="err" role="alert">{{ errors.instagram }}</span>
                      </div>
                      <div class="affix">
                        <span class="affix__at" aria-hidden="true">@</span>
                        <input
                          id="f-ig"
                          ref="igInput"
                          v-model="form.instagram"
                          type="text"
                          inputmode="text"
                          autocomplete="off"
                          autocapitalize="none"
                          autocorrect="off"
                          spellcheck="false"
                          maxlength="30"
                          placeholder="username"
                          class="input"
                          :class="{ shake: shaking && errors.instagram }"
                          :aria-invalid="!!errors.instagram"
                          @focus="focusField = 'instagram'"
                          @blur="focusField = 'idle'"
                          @input="cleanHandle('instagram')"
                        />
                      </div>
                    </div>

                    <!-- TikTok -->
                    <div class="field">
                      <div class="label">
                        <label for="f-tt">TikTok (Opsional)</label>
                        <span v-if="errors.tiktok" class="err" role="alert">{{ errors.tiktok }}</span>
                      </div>
                      <div class="affix">
                        <span class="affix__at" aria-hidden="true">@</span>
                        <input
                          id="f-tt"
                          ref="ttInput"
                          v-model="form.tiktok"
                          type="text"
                          inputmode="text"
                          autocomplete="off"
                          autocapitalize="none"
                          autocorrect="off"
                          spellcheck="false"
                          maxlength="24"
                          placeholder="username"
                          class="input"
                          :class="{ shake: shaking && errors.tiktok }"
                          :aria-invalid="!!errors.tiktok"
                          @focus="focusField = 'tiktok'"
                          @blur="focusField = 'idle'"
                          @input="cleanHandle('tiktok')"
                        />
                      </div>
                    </div>
                  </div>

                  <!-- Pilihan cara kirim -->
                  <div class="toggle" role="group" aria-label="Cara mengirim cerita">
                    <span
                      class="toggle__pill"
                      :class="inputType === 'text' ? 'toggle__pill--text' : 'toggle__pill--file'"
                      aria-hidden="true"
                    ></span>
                    <button
                      type="button"
                      class="toggle__btn"
                      :class="{ 'is-active': inputType === 'text' }"
                      :aria-pressed="inputType === 'text'"
                      @click="inputType = 'text'"
                    >
                      Ketik Manual
                    </button>
                    <button
                      type="button"
                      class="toggle__btn"
                      :class="{ 'is-active': inputType === 'file' }"
                      :aria-pressed="inputType === 'file'"
                      @click="inputType = 'file'"
                    >
                      Unggah Dokumen
                    </button>
                  </div>

                  <!-- Isi cerita -->
                  <div class="content">
                    <div class="content__area">
                      <textarea
                        v-if="inputType === 'text'"
                        v-model="form.text"
                        :maxlength="MAX_CHARS"
                        placeholder="Malam itu terdengar langkah kaki di atap rumah, padahal tidak ada tangga menuju ke sana..."
                        aria-label="Isi cerita"
                        class="input textarea custom-scrollbar"
                        :class="{ shake: shaking && errors.content }"
                        :aria-invalid="!!errors.content"
                        @focus="focusField = 'text'"
                        @blur="focusField = 'idle'"
                        @input="errors.content = ''"
                      ></textarea>

                      <div
                        v-else
                        class="drop"
                        :class="{ 'drop--over': isDragging, 'drop--filled': !!file, shake: shaking && errors.content }"
                        role="button"
                        tabindex="0"
                        :aria-label="file ? 'Ganti file cerita' : 'Pilih file cerita PDF atau DOCX'"
                        @click="openPicker"
                        @keydown.enter.prevent="openPicker"
                        @keydown.space.prevent="openPicker"
                        @focus="focusField = 'file'"
                        @blur="focusField = 'idle'"
                        @dragenter.prevent="isDragging = true"
                        @dragover.prevent="isDragging = true"
                        @dragleave.prevent="isDragging = false"
                        @drop.prevent="onDrop"
                      >
                        <input
                          ref="fileInput"
                          type="file"
                          class="sr-only"
                          tabindex="-1"
                          accept=".pdf,.docx,application/pdf,application/vnd.openxmlformats-officedocument.wordprocessingml.document"
                          @change="onPick"
                        />
                        <template v-if="file">
                          <span class="drop__icon" aria-hidden="true">📄</span>
                          <p class="drop__name">{{ file.name }}</p>
                          <p class="drop__meta">{{ formatSize(file.size) }} · klik untuk ganti</p>
                          <button type="button" class="drop__remove" @click.stop="removeFile">Hapus file</button>
                        </template>
                        <template v-else>
                          <span class="drop__icon" aria-hidden="true">📄</span>
                          <p class="drop__title">{{ isDragging ? 'Lepaskan file di sini' : 'Seret File PDF/DOCX ke sini' }}</p>
                          <p class="drop__meta">atau klik untuk memilih · maks. {{ MAX_FILE_MB }} MB</p>
                        </template>
                      </div>
                    </div>

                    <div class="content__meta">
                      <span v-if="errors.content" class="err" role="alert">{{ errors.content }}</span>
                      <span v-else class="meta-spacer"></span>
                      <span
                        v-if="inputType === 'text'"
                        class="counter"
                        :class="{ 'counter--warn': charRatio > 0.9, 'counter--max': charRatio >= 1 }"
                      >
                        {{ form.text.length }} / {{ MAX_CHARS }}
                      </span>
                    </div>
                  </div>

                  <!-- Tombol kirim: permen/lendir dengan kelelawar -->
                  <button type="submit" class="submit" :disabled="isSubmitting">
                    <svg class="submit__bat" viewBox="0 0 64 32" aria-hidden="true">
                      <path d="M32 6 L35 11 Q42 4 62 8 Q54 12 54 20 Q50 16 46 19 Q43 15 39 20 Q36 17 32 27 Q28 17 25 20 Q21 15 18 19 Q14 16 10 20 Q10 12 2 8 Q22 4 29 11 Z" />
                    </svg>
                    <span v-if="isSubmitting" class="spinner" aria-hidden="true"></span>
                    <span class="submit__label">{{ isSubmitting ? 'MENGIRIM...' : 'KIRIM CERITA SERAM!' }}</span>
                    <svg class="submit__bat" viewBox="0 0 64 32" aria-hidden="true">
                      <path d="M32 6 L35 11 Q42 4 62 8 Q54 12 54 20 Q50 16 46 19 Q43 15 39 20 Q36 17 32 27 Q28 17 25 20 Q21 15 18 19 Q14 16 10 20 Q10 12 2 8 Q22 4 29 11 Z" />
                    </svg>
                  </button>
                </form>

                <!-- Halaman sukses -->
                <div v-else key="success" class="success">
                  <h2 class="success__title">TERIMA KASIH!</h2>
                  <p class="success__text">
                    Bisikanmu sudah kami terima<template v-if="form.name.trim()">, {{ form.name.trim() }}</template>.
                    Kalau ceritamu terpilih, kami akan memberi kabar lewat akun yang kamu cantumkan.
                  </p>
                  <button type="button" class="submit submit--pill" @click="resetForm">
                    <span class="submit__label">KIRIM CERITA LAIN</span>
                  </button>
                </div>
              </Transition>
            </div>
          </div>

          <!-- Maskot + gelembung ucapan -->
          <div class="buddy" :class="wide ? 'buddy--wide' : 'buddy--col'">
            <div class="bubble" :class="wide ? 'bubble--wide' : 'bubble--col'" role="status" aria-live="polite">
              <Transition name="swap" mode="out-in">
                <span :key="hint">{{ hint }}</span>
              </Transition>
            </div>
            <button
              type="button"
              class="mascot"
              :class="['mascot--' + mood, { 'mascot--poke': poking }]"
              aria-label="Sapa maskot"
              @click="poke"
            >
              <img :src="mascotImg" alt="Maskot penyihir" draggable="false" />
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, watch, onMounted, onUnmounted, nextTick } from 'vue'

// Pastikan alias '@' mengarah ke 'src' (default di proyek Vite + Vue).
import bgMain from '@/assets/bg-main.png'
import frameImg from '@/assets/frame.png'
import titleImg from '@/assets/title.png'
import slimeSide from '@/assets/slime-side.png'
import mascotImg from '@/assets/mascot.png'
import logoImg from '@/assets/logo.png'

type Screen = 'loading' | 'form' | 'success'
type Handle = 'instagram' | 'tiktok'
type Focus = 'idle' | 'name' | Handle | 'text' | 'file'

const MAX_CHARS = 5000
const MIN_CHARS = 30
// Samakan dengan backend (4 MB)
const MAX_FILE_MB = 4
const API_URL = 'https://backend-creepypasta.vercel.app/api/submit'
// Username IG: huruf, angka, titik, underscore (maks. 30). TikTok sama (maks. 24).
const HANDLE_RE = /^[A-Za-z0-9._]+$/

// ---------- State ----------
const currentScreen = ref<Screen>('loading')
const inputType = ref<'text' | 'file'>('text')
const form = reactive({ name: '', instagram: '', tiktok: '', text: '' })
const file = ref<File | null>(null)
const errors = reactive({ name: '', instagram: '', tiktok: '', content: '' })
const shaking = ref(false)
const isSubmitting = ref(false)
const isDragging = ref(false)
const focusField = ref<Focus>('idle')
const progress = ref(0)

const nameInput = ref<HTMLInputElement | null>(null)
const igInput = ref<HTMLInputElement | null>(null)
const ttInput = ref<HTMLInputElement | null>(null)
const fileInput = ref<HTMLInputElement | null>(null)

// ---------- Ukuran layar ----------
// "wide"  : bingkai 16:9 asli (desktop / tablet landscape) yang diskalakan utuh
// "col"   : kolom yang bisa di-scroll dengan papan CSS (HP / portrait)
const viewport = reactive({ w: 1280, h: 720 })
const wide = computed(() => viewport.w >= 900 && viewport.h >= 520 && viewport.w / viewport.h >= 1.2)
const stageScale = computed(() => Math.min(viewport.w / 1280, viewport.h / 720))
const stageStyle = computed(() => ({
  transform: `translateX(-50%) scale(${stageScale.value})`,
}))
const updateViewport = () => {
  viewport.w = window.innerWidth
  viewport.h = window.innerHeight
}

// ---------- Loading: progres asli dari pemuatan gambar + font ----------
const preloadAll = () => {
  const sources = [bgMain, frameImg, titleImg, slimeSide, mascotImg, logoImg]
  const total = sources.length + 1 // +1 untuk font
  let done = 0
  const tick = () => {
    done += 1
    progress.value = Math.max(progress.value, (done / total) * 100)
  }

  const loads = sources.map(
    (src) =>
      new Promise<void>((resolve) => {
        const img = new Image()
        img.onload = img.onerror = () => {
          tick()
          resolve()
        }
        img.src = src
      }),
  )

  // Tunggu font siap supaya teks tidak "loncat" saat form muncul (maks. 4 detik).
  const fonts = Promise.race([
    Promise.all([
      document.fonts.load('1em CreepyFont', 'Aa'),
      document.fonts.load('500 1em "Baloo 2"', '0123456789'), // font cadangan untuk angka
    ]),
    new Promise<void>((resolve) => setTimeout(resolve, 4000)),
  ])
    .catch(() => undefined)
    .then(tick)

  const minDelay = new Promise<void>((resolve) => setTimeout(resolve, 1400))
  return Promise.all([...loads, fonts, minDelay])
}

onMounted(async () => {
  updateViewport()
  window.addEventListener('resize', updateViewport)
  await preloadAll()
  progress.value = 100
  setTimeout(() => (currentScreen.value = 'form'), 250)
})

onUnmounted(() => {
  window.removeEventListener('resize', updateViewport)
  if (pokeTimer) clearTimeout(pokeTimer)
})

// ---------- Maskot ----------
const pokeLines = [
  'Hihi, geli!',
  'Jangan dipencet terus, nanti kukutuk jadi kodok.',
  'Ayo, ceritanya jangan ditahan-tahan.',
]
const pokeLine = ref('')
const poking = ref(false)
let pokeTimer: ReturnType<typeof setTimeout> | undefined
let pokeIndex = 0

const poke = () => {
  pokeLine.value = pokeLines[pokeIndex++ % pokeLines.length] ?? ''
  poking.value = true
  if (pokeTimer) clearTimeout(pokeTimer)
  pokeTimer = setTimeout(() => {
    poking.value = false
    pokeLine.value = ''
  }, 2600)
}

const hasError = computed(() => !!(errors.name || errors.instagram || errors.tiktok || errors.content))

// Suasana maskot: melompat senang saat sukses, gelisah saat ada error
const mood = computed<'calm' | 'worry' | 'happy'>(() => {
  if (currentScreen.value === 'success') return 'happy'
  if (hasError.value) return 'worry'
  return 'calm'
})

const hint = computed(() => {
  if (currentScreen.value === 'success') return 'Bisikanmu sudah sampai. Sampai jumpa di cerita berikutnya!'
  if (isSubmitting.value) return 'Menyampaikan bisikanmu...'
  if (hasError.value) return 'Ups, ada isian yang perlu dicek lagi.'
  if (pokeLine.value) return pokeLine.value
  switch (focusField.value) {
    case 'name':
      return 'Pakai nama samaran juga boleh, kok.'
    case 'instagram':
      return 'Opsional. Tulis username-nya saja, tanpa @.'
    case 'tiktok':
      return 'Opsional juga. Isi salah satu atau dua-duanya.'
    case 'text':
      return 'Makin detail, makin merinding.'
    case 'file':
      return `PDF atau DOCX, maksimal ${MAX_FILE_MB} MB.`
    default:
      return 'Hai! Bisikkan ceritamu padaku...'
  }
})

// ---------- Cerita ----------
const charRatio = computed(() => form.text.length / MAX_CHARS)

watch(inputType, () => {
  errors.content = ''
})

// ---------- Username sosmed ----------
// Buang spasi dan "@" (sudah ada awalan @ di kolomnya), jadi paste "@nama" tetap rapi.
const cleanHandle = (key: Handle) => {
  form[key] = form[key].replace(/[\s@]/g, '')
  errors[key] = ''
}

// ---------- File ----------
const ALLOWED_EXT = ['.pdf', '.docx']
const formatSize = (bytes: number) =>
  bytes < 1024 * 1024 ? `${Math.max(1, Math.round(bytes / 1024))} KB` : `${(bytes / 1024 / 1024).toFixed(1)} MB`

const acceptFile = (f?: File | null) => {
  if (!f) return
  const ok = ALLOWED_EXT.some((ext) => f.name.toLowerCase().endsWith(ext))
  if (!ok) {
    file.value = null
    errors.content = 'Format harus PDF atau DOCX.'
    return
  }
  if (f.size > MAX_FILE_MB * 1024 * 1024) {
    file.value = null
    errors.content = `Ukuran file maksimal ${MAX_FILE_MB} MB.`
    return
  }
  file.value = f
  errors.content = ''
}

const openPicker = () => fileInput.value?.click()
const onPick = (e: Event) => {
  const input = e.target as HTMLInputElement
  acceptFile(input.files?.[0])
  input.value = '' // izinkan memilih file yang sama lagi
}
const onDrop = (e: DragEvent) => {
  isDragging.value = false
  acceptFile(e.dataTransfer?.files?.[0])
}
const removeFile = () => {
  file.value = null
}

// ---------- Fungsi Bantuan: File ke Base64 ----------
const fileToBase64 = (f: File): Promise<string> =>
  new Promise((resolve, reject) => {
    const reader = new FileReader()
    reader.onload = () => {
      // Hapus prefix "data:application/pdf;base64," dan ambil string aslinya saja
      const base64String = (reader.result as string).split(',')[1] ?? ''
      if (base64String) resolve(base64String)
      else reject(new Error('File tidak bisa dibaca. Coba pilih file lain.'))
    }
    reader.onerror = () => reject(new Error('File tidak bisa dibaca. Coba pilih file lain.'))
    reader.readAsDataURL(f)
  })

// ---------- Kirim ----------
const validate = () => {
  errors.name = form.name.trim() ? '' : 'Wajib diisi.'
  ;(['instagram', 'tiktok'] as const).forEach((key) => {
    errors[key] = !form[key] || HANDLE_RE.test(form[key]) ? '' : 'Tidak valid.'
  })
  if (inputType.value === 'text') {
    const len = form.text.trim().length
    errors.content = len >= MIN_CHARS ? '' : `Minimal ${MIN_CHARS} karakter (sekarang ${len}).`
  } else if (!file.value) {
    errors.content = errors.content || 'Pilih file PDF atau DOCX dulu.'
  }
  return !hasError.value
}

const submitStory = async () => {
  if (isSubmitting.value) return
  if (!validate()) {
    shaking.value = true
    setTimeout(() => (shaking.value = false), 500)
    await nextTick()
    if (errors.name) nameInput.value?.focus()
    else if (errors.instagram) igInput.value?.focus()
    else if (errors.tiktok) ttInput.value?.focus()
    return
  }
  isSubmitting.value = true

  try {
    // Struktur data JSON sesuai permintaan backend Drizzle
    const payload: Record<string, string | null> = {
      authorName: form.name.trim(),
      instagram: form.instagram ? `@${form.instagram}` : null,
      tiktok: form.tiktok ? `@${form.tiktok}` : null,
    }

    if (inputType.value === 'text') {
      payload.storyText = form.text.trim()
    } else if (file.value) {
      payload.fileName = file.value.name
      payload.fileBase64 = await fileToBase64(file.value)
    }

    // Panggil API Vercel
    const response = await fetch(API_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(payload),
    })

    // Balasan bisa bukan JSON (mis. 413 dari Vercel), jadi jangan langsung .json()
    const data = await response.json().catch(() => null)

    // Tangkap error dari backend jika sukses: false
    if (!response.ok || !data?.success) {
      const tooBig = response.status === 413
      throw new Error(
        data?.message ||
          (tooBig ? 'File terlalu besar untuk dikirim. Coba file yang lebih kecil.' : 'Terjadi kesalahan pada server.'),
      )
    }

    currentScreen.value = 'success'
  } catch (error) {
    console.error('Error submit:', error)
    // Gagal fetch (offline / CORS) berupa TypeError berbahasa Inggris, jadi diganti pesan sendiri
    const isNetworkError = error instanceof TypeError
    errors.content =
      !isNetworkError && error instanceof Error && error.message
        ? error.message
        : 'Gagal terhubung ke server. Coba lagi nanti.'
    shaking.value = true
    setTimeout(() => (shaking.value = false), 500)
  } finally {
    isSubmitting.value = false
  }
}

const resetForm = () => {
  form.name = ''
  form.instagram = ''
  form.tiktok = ''
  form.text = ''
  file.value = null
  inputType.value = 'text'
  errors.name = ''
  errors.instagram = ''
  errors.tiktok = ''
  errors.content = ''
  currentScreen.value = 'form'
}
</script>

<style>
/* Font cadangan. CreepyFont (versi demo) mengganti angka, ":" dan "*" dengan watermark,
   jadi karakter itu otomatis memakai Baloo 2 lewat unicode-range di main.css. */
@import url('https://fonts.googleapis.com/css2?family=Baloo+2:wght@500&display=swap');
</style>

<style scoped>
/* ================= Dasar ================= */
.app {
  --font-ui: 'CreepyFont', 'Baloo 2', ui-rounded, system-ui, sans-serif;
  /* Kalau teks cerita di kolom ketik terasa terlalu sulit dibaca, ganti ke:
     --font-read: 'Baloo 2', ui-rounded, system-ui, sans-serif; */
  --font-read: var(--font-ui);

  position: relative;
  width: 100%;
  font-family: var(--font-ui);
  font-weight: 400;
  font-synthesis: none; /* font dekoratif hanya punya 1 ketebalan: jangan dipalsukan jadi bold */
  color: #fed7aa;
  background: #2e0a4d;
}
.app--wide {
  height: 100dvh;
  overflow: hidden;
}
.app--col {
  min-height: 100dvh;
  overflow-x: hidden;
}
.app *,
.app *::before,
.app *::after {
  box-sizing: border-box;
}
.app h2,
.app p {
  margin: 0;
}
.app__bg {
  position: fixed;
  /* sedikit melebar ke kiri: tepi kiri gambar asli punya garis gelap tipis */
  inset: 0 0 0 -14px;
  background-size: cover;
  background-repeat: no-repeat;
  z-index: 0;
}
.main {
  position: relative;
  z-index: 1;
}
.app--wide .main {
  position: absolute;
  inset: 0;
}
.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  overflow: hidden;
  clip: rect(0 0 0 0);
  white-space: nowrap;
}

/* ================= Loading ================= */
.loading {
  position: relative;
  z-index: 1;
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 24px;
}
.loading__bar {
  margin-top: 40px;
  width: min(20rem, 80vw);
  height: 1.5rem;
  border: 2px solid #ec4899;
  border-radius: 9999px;
  padding: 4px;
  background: rgba(59, 7, 100, 0.5);
}
.loading__fill {
  height: 100%;
  border-radius: 9999px;
  background: linear-gradient(90deg, #f97316, #ec4899);
  transition: width 0.35s ease-out;
}
.loading__text {
  margin-top: 16px;
  color: #fb923c;
  font-size: 1.75rem;
  letter-spacing: 0.08em;
  text-shadow: 0 2px 4px rgba(0, 0, 0, 0.4);
}

/* ================= Logo ================= */
/* Logo bernuansa marun, jadi diberi lencana pink senada lendir agar kontras di latar ungu */
.logo-badge {
  display: grid;
  place-items: center;
  background: linear-gradient(160deg, #ffc2ff, #f08cf0);
  border: 2px solid #ffe0ff;
  box-shadow: 0 0 18px rgba(236, 72, 153, 0.6);
}
.logo-badge img {
  display: block;
  width: 62%;
  height: auto;
}
.logo-badge--xl {
  width: 9.5rem;
  height: 9.5rem;
  border-radius: 2rem;
}
.logo-badge--wide {
  position: absolute;
  left: 34px;
  top: 24px;
  width: 92px;
  height: 92px;
  border-radius: 22px;
  z-index: 5;
}
/* Mobile: pojok kiri atas, sama seperti di desktop (tidak lagi menumpuk di atas judul) */
.logo-badge--col {
  position: absolute;
  left: 14px;
  top: 12px;
  width: 2.9rem;
  height: 2.9rem;
  border-radius: 0.9rem;
  z-index: 5;
}

/* ================= Panggung ================= */
.stage {
  position: relative;
}
.stage--wide {
  position: absolute;
  left: 50%;
  bottom: 0;
  width: 1280px;
  height: 720px;
  transform-origin: 50% 100%;
}
.stage--col {
  min-height: 100dvh;
  display: flex;
  flex-direction: column;
  align-items: center;
  /* atas: ruang untuk logo. bawah: maskot berdiri di atas lendir latar */
  padding: 3.6rem 16px calc(env(safe-area-inset-bottom, 0px) + max(4vh, 26px));
}
.frame {
  position: absolute;
  inset: 0;
  width: 1280px;
  height: 720px;
  z-index: 1;
  pointer-events: none;
  filter: drop-shadow(0 0 26px rgba(236, 72, 153, 0.28));
}
.slime {
  position: absolute;
  bottom: 0;
  width: 1280px;
  height: 135px;
  z-index: 0;
  pointer-events: none;
}
.slime--left {
  left: -1280px;
}
.slime--right {
  left: 1280px;
}

/* Judul + kartu */
.group--wide {
  display: contents;
}
.group--col {
  width: 100%;
  max-width: 460px;
  margin-block: auto; /* kartu berada di tengah ruang yang tersisa */
  display: flex;
  flex-direction: column;
  align-items: center;
}
.title {
  position: relative;
  z-index: 3;
  width: min(84vw, 420px);
  margin-bottom: -1.7rem;
  pointer-events: none;
  filter: drop-shadow(0 4px 10px rgba(30, 0, 50, 0.45));
}

/* ================= Kartu ================= */
.card {
  position: relative;
  z-index: 2;
}
/* Desktop: area aman di dalam papan kosong pada bingkai (koordinat 1280x720).
   Papan berisi x 215-1062, jadi kartu dipusatkan di x = 640. */
.card--wide {
  position: absolute;
  left: 240px;
  top: 232px;
  width: 800px;
  height: 322px;
}
/* Mobile: papan digambar dengan CSS memakai warna yang diambil dari bingkai */
.card--col {
  width: 100%;
  padding: 2.4rem 1.1rem 1.3rem;
  background:
    linear-gradient(180deg, rgba(255, 255, 255, 0.04), transparent 30%),
    #65138c;
  border: 5px solid #9e47c0;
  border-radius: 0.9rem;
  box-shadow:
    0 0 0 2px #dc78e4,
    inset 0 0 0 4px #3e0062,
    0 0 30px rgba(236, 72, 153, 0.35);
}

/* ================= Form ================= */
.story-form {
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
}
.card--wide .story-form,
.card--wide .success {
  height: 100%;
}
.card--wide .story-form {
  gap: 9px;
}

/* Mobile: nama 1 baris penuh, Instagram + TikTok berdampingan */
.field-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 0.55rem 0.75rem;
}
.field {
  min-width: 0;
}
.field--name {
  grid-column: 1 / -1;
}
/* Desktop: ketiganya sebaris */
.card--wide .field-row {
  grid-template-columns: 1.45fr 1fr 1fr;
  gap: 16px;
}
.card--wide .field--name {
  grid-column: auto;
}

.label {
  display: flex;
  flex-wrap: wrap;
  justify-content: space-between;
  align-items: baseline;
  gap: 0 0.5rem;
  min-height: 1.3rem;
  margin-bottom: 0.15rem;
  color: #fb923c;
  font-size: 1rem;
  line-height: 1.25;
  text-shadow: 0 1px 2px rgba(0, 0, 0, 0.45);
}
.err {
  color: #f9a8d4;
  font-size: 0.9rem;
  line-height: 1.2;
}
.input {
  width: 100%;
  background: rgba(59, 7, 100, 0.6);
  color: #fed7aa;
  border: 1px solid rgba(249, 115, 22, 0.8);
  border-radius: 0.5rem;
  padding: 0.55rem 0.8rem;
  font: inherit;
  font-size: 1.1rem; /* >= 16px: mencegah zoom otomatis di iOS */
  line-height: 1.25;
  outline: none;
  backdrop-filter: blur(4px);
  transition: border-color 0.15s, box-shadow 0.15s;
}
.card--wide .input {
  padding: 0.28rem 0.75rem;
}
.input::placeholder {
  color: #c084fc;
}
.input:focus {
  border-color: #ec4899;
  box-shadow: 0 0 0 3px rgba(236, 72, 153, 0.3);
}
.input[aria-invalid='true'] {
  border-color: #f472b6;
  box-shadow: 0 0 0 2px rgba(244, 114, 182, 0.35);
}

/* Awalan @ untuk Instagram / TikTok */
.affix {
  position: relative;
}
.affix__at {
  position: absolute;
  left: 0.8rem;
  top: 50%;
  transform: translateY(-50%);
  color: #fb923c;
  font-size: 1.1rem;
  line-height: 1;
  pointer-events: none;
}
.affix .input {
  padding-left: 1.9rem;
}

/* Pilihan cara kirim */
.toggle {
  position: relative;
  display: grid;
  grid-template-columns: 1fr 1fr;
  padding: 4px;
  background: rgba(59, 7, 100, 0.8);
  border: 1px solid #6b21a8;
  border-radius: 0.5rem;
}
.toggle__pill {
  position: absolute;
  top: 4px;
  bottom: 4px;
  left: 4px;
  width: calc(50% - 4px);
  border-radius: 0.375rem;
  transition: transform 0.3s cubic-bezier(0.3, 0.9, 0.3, 1), background 0.3s, box-shadow 0.3s;
}
.toggle__pill--text {
  background: linear-gradient(90deg, #ea580c, #f97316);
  box-shadow: 0 0 10px rgba(234, 88, 12, 0.8);
}
.toggle__pill--file {
  transform: translateX(100%);
  background: linear-gradient(90deg, #db2777, #9333ea);
  box-shadow: 0 0 10px rgba(236, 72, 153, 0.8);
}
.toggle__btn {
  position: relative;
  z-index: 1;
  padding: 0.45rem 0.25rem;
  border-radius: 0.375rem;
  color: #c084fc;
  font: inherit;
  font-size: 1.1rem;
  line-height: 1.25;
  background: transparent;
  cursor: pointer;
  transition: color 0.2s;
}
.toggle__btn.is-active {
  color: #fff;
}
.card--wide .toggle__btn {
  padding: 0.12rem 0.25rem;
}

/* Isi cerita */
.content {
  display: flex;
  flex-direction: column;
  gap: 0.2rem;
}
.card--wide .content {
  flex: 1;
  min-height: 0;
}
.content__area {
  position: relative;
  min-height: 10rem;
}
.card--wide .content__area {
  flex: 1;
  min-height: 0;
}
.textarea {
  position: absolute;
  inset: 0;
  height: 100%;
  resize: none;
  font-family: var(--font-read);
  line-height: 1.5;
}
.card--wide .textarea {
  padding: 0.5rem 0.75rem;
}
.content__meta {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  gap: 0.75rem;
  min-height: 1.1rem;
}
.counter {
  margin-left: auto;
  white-space: nowrap;
  color: #c084fc;
  font-size: 0.95rem;
  font-variant-numeric: tabular-nums;
}
.counter--warn {
  color: #fb923c;
}
.counter--max {
  color: #f472b6;
}

/* Zona unggah */
.drop {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.15rem;
  padding: 0.5rem;
  text-align: center;
  border: 2px dashed #ec4899;
  border-radius: 0.5rem;
  background: rgba(59, 7, 100, 0.6);
  cursor: pointer;
  outline: none;
  transition: background 0.2s, border-color 0.2s, transform 0.2s;
}
.drop:hover,
.drop:focus-visible {
  background: rgba(157, 23, 77, 0.35);
}
.drop:focus-visible {
  box-shadow: 0 0 0 3px rgba(236, 72, 153, 0.4);
}
.drop--over {
  background: rgba(190, 24, 93, 0.45);
  border-color: #fb923c;
  transform: scale(1.01);
}
.drop--filled {
  border-style: solid;
  border-color: #fb923c;
}
.drop__icon {
  font-size: 1.75rem;
  line-height: 1.1;
}
.drop__title,
.drop__name {
  color: #f9a8d4;
  font-size: 1.15rem;
}
.drop__name {
  max-width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: #fed7aa;
}
.drop__meta {
  color: #c084fc;
  font-size: 0.95rem;
}
.drop__remove {
  margin-top: 0.15rem;
  padding: 0.1rem 0.8rem;
  border: 1px solid rgba(244, 114, 182, 0.7);
  border-radius: 9999px;
  color: #f9a8d4;
  font: inherit;
  font-size: 0.95rem;
  background: transparent;
  cursor: pointer;
}
.drop__remove:hover {
  background: rgba(244, 114, 182, 0.2);
}

/* ================= Tombol kirim ================= */
/* Gaya mengikuti judul: isi oranye ke pink, garis luar ungu tua, cincin pink muda, dasar 3D */
.submit {
  --base: #7a1b8f;
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  gap: 0.55em;
  width: 100%;
  margin-bottom: 8px; /* ruang untuk dasar 3D */
  padding: 0.5rem 1rem 0.65rem;
  border: 3px solid #3b0764;
  border-radius: 9999px;
  overflow: hidden;
  color: #fff7ed;
  font: inherit;
  font-size: clamp(1.25rem, 5.6vw, 1.6rem);
  line-height: 1.1;
  letter-spacing: 0.08em;
  white-space: nowrap;
  text-shadow:
    0 2px 0 #8a1c8f,
    0 -1px 0 #8a1c8f,
    1px 0 0 #8a1c8f,
    -1px 0 0 #8a1c8f,
    0 0 14px rgba(255, 224, 255, 0.55);
  background: linear-gradient(180deg, #ffb347 0%, #f97316 42%, #ec4899 100%);
  box-shadow:
    0 0 0 2px #ffc2ff,
    0 6px 0 2px var(--base),
    0 12px 22px rgba(236, 72, 153, 0.5);
  cursor: pointer;
  transition: transform 0.15s ease, box-shadow 0.15s ease, filter 0.2s;
}
.card--wide .submit {
  padding: 0.2rem 1rem 0.32rem;
  font-size: 1.5rem;
}
/* kilap */
.submit::before {
  content: '';
  position: absolute;
  left: 7%;
  right: 7%;
  top: 3px;
  height: 42%;
  border-radius: 9999px;
  background: linear-gradient(180deg, rgba(255, 255, 255, 0.55), rgba(255, 255, 255, 0.04));
  pointer-events: none;
}
/* kilatan yang lewat sesekali */
.submit::after {
  content: '';
  position: absolute;
  top: 0;
  bottom: 0;
  left: -40%;
  width: 28%;
  background: linear-gradient(100deg, transparent, rgba(255, 255, 255, 0.5), transparent);
  transform: skewX(-20deg);
  animation: shimmer 5s ease-in-out infinite;
  pointer-events: none;
}
.submit__label,
.submit__bat,
.spinner {
  position: relative;
  z-index: 1;
}
.submit__bat {
  flex: none;
  width: 1.3em;
  height: auto;
  fill: #3b0764;
  filter: drop-shadow(0 1px 0 rgba(255, 224, 255, 0.65));
}
.submit:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow:
    0 0 0 2px #ffc2ff,
    0 8px 0 2px var(--base),
    0 16px 28px rgba(236, 72, 153, 0.7);
}
.submit:hover:not(:disabled) .submit__bat {
  animation: flap 0.28s ease-in-out infinite alternate;
}
.submit:active:not(:disabled) {
  transform: translateY(5px);
  box-shadow:
    0 0 0 2px #ffc2ff,
    0 1px 0 2px var(--base),
    0 4px 10px rgba(236, 72, 153, 0.5);
}
.submit:focus-visible {
  outline: 3px solid #fed7aa;
  outline-offset: 4px;
}
.submit:disabled {
  cursor: progress;
  filter: saturate(0.7) brightness(0.9);
}
.submit--pill {
  width: auto;
  min-width: 14rem;
  padding-inline: 1.6rem;
  font-size: 1.35rem;
}
.card--wide .submit--pill {
  font-size: 1.4rem;
}
.spinner {
  width: 1em;
  height: 1em;
  border: 3px solid rgba(255, 255, 255, 0.4);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.8s linear infinite;
}

/* Sukses */
.success {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  gap: 0.75rem;
  padding: 0.5rem 0 1rem;
  text-align: center;
}
.card--col .success {
  min-height: 16rem;
}
.success__title {
  color: #fb923c;
  font-size: 2.8rem;
  font-weight: 400;
  line-height: 1;
  text-shadow: 0 2px 6px rgba(0, 0, 0, 0.45), 0 0 18px rgba(249, 115, 22, 0.5);
}
.card--wide .success__title {
  font-size: 3.6rem;
}
.success__text {
  max-width: 32rem;
  color: #fed7aa;
  font-size: 1.2rem;
  line-height: 1.5;
}

/* ================= Maskot & gelembung ================= */
.buddy {
  position: relative;
  z-index: 4;
}
/* Desktop: berdiri di pojok kanan bawah panggung, "tenggelam" di lendir bingkai.
   Gelembung ada di atas kepalanya dan ekornya menunjuk ke topi. */
.buddy--wide {
  position: absolute;
  right: 4px;
  bottom: 14px;
  width: 214px;
  display: flex;
  flex-direction: column;
  align-items: center;
}
/* Mobile: satu baris di bawah kartu, berdiri di atas lendir latar */
.buddy--col {
  width: 100%;
  max-width: 460px;
  margin-top: 0.75rem;
  display: flex;
  align-items: flex-end;
  gap: 0.6rem;
  padding-inline: 0.25rem;
}

.mascot {
  display: block;
  padding: 0;
  background: none;
  border: 0;
  cursor: pointer;
  transform-origin: 50% 100%;
  animation: float 3.4s ease-in-out infinite;
  filter: drop-shadow(0 0 12px rgba(236, 72, 153, 0.7));
  -webkit-tap-highlight-color: transparent;
}
.buddy--wide .mascot {
  width: 100%;
}
.buddy--col .mascot {
  flex: none;
  width: 5.6rem;
}
.mascot img {
  display: block;
  width: 100%;
  height: auto;
  pointer-events: none;
  /* bagian bawah memudar ke lendir, jadi tidak terlihat terpotong rata */
  -webkit-mask-image: linear-gradient(to bottom, #000 78%, transparent 100%);
  mask-image: linear-gradient(to bottom, #000 78%, transparent 100%);
}
.mascot:focus-visible {
  outline: 3px solid #fed7aa;
  outline-offset: 4px;
  border-radius: 1rem;
}
.mascot--happy {
  animation: hop 1.1s ease-in-out infinite;
}
.mascot--worry {
  animation: fret 0.5s ease-in-out 2, float 3.4s ease-in-out 1s infinite;
}
.mascot--poke {
  animation: wiggle 0.6s ease-in-out;
}

.bubble {
  position: relative;
  padding: 0.5rem 0.8rem;
  color: #3b0764;
  font-size: 1.05rem;
  line-height: 1.25;
  text-align: center;
  background: #ffe0ff;
  border: 2px solid #f08cf0;
  border-radius: 0.9rem;
  box-shadow: 0 4px 14px rgba(30, 0, 50, 0.35);
  pointer-events: none;
}
.bubble::after {
  content: '';
  position: absolute;
  width: 12px;
  height: 12px;
  background: #ffe0ff;
  border-right: 2px solid #f08cf0;
  border-bottom: 2px solid #f08cf0;
}
.bubble--wide {
  width: 100%;
  margin-bottom: 12px;
  font-size: 1.1rem;
}
.bubble--wide::after {
  left: 50%;
  bottom: -8px;
  margin-left: -6px;
  transform: rotate(45deg);
}
.bubble--col {
  flex: 1;
  min-height: 3.4rem;
  display: grid;
  place-items: center;
  margin-bottom: 1.5rem;
}
.bubble--col::after {
  right: -8px;
  top: 50%;
  margin-top: -6px;
  transform: rotate(-45deg);
}

/* ================= Tablet portrait ================= */
@media (min-width: 600px) {
  .group--col,
  .buddy--col {
    max-width: 560px;
  }
  .title {
    width: 500px;
    margin-bottom: -2rem;
  }
  .card--col {
    padding: 2.8rem 1.6rem 1.6rem;
  }
  .buddy--col .mascot {
    width: 7rem;
  }
}
/* HP sangat kecil: kelelawar di tombol disembunyikan agar teks tidak turun baris */
@media (max-width: 339px) {
  .submit__bat {
    display: none;
  }
}

/* ================= Animasi ================= */
@keyframes float {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-6px); }
}
@keyframes wiggle {
  0%, 100% { transform: rotate(0) scale(1); }
  25% { transform: rotate(-7deg) scale(1.05); }
  60% { transform: rotate(6deg) scale(1.05); }
}
@keyframes hop {
  0%, 100% { transform: translateY(0) scale(1, 1); }
  30% { transform: translateY(-16px) scale(0.97, 1.04); }
  55% { transform: translateY(0) scale(1.04, 0.96); }
}
@keyframes fret {
  0%, 100% { transform: rotate(0); }
  25% { transform: rotate(-4deg); }
  75% { transform: rotate(4deg); }
}
@keyframes spin {
  to { transform: rotate(360deg); }
}
@keyframes glow {
  0%, 100% { transform: scale(1); box-shadow: 0 0 22px rgba(236, 72, 153, 0.6); }
  50% { transform: scale(1.05); box-shadow: 0 0 44px rgba(236, 72, 153, 0.95); }
}
@keyframes shake {
  0%, 100% { transform: translateX(0); }
  20% { transform: translateX(-6px); }
  40% { transform: translateX(6px); }
  60% { transform: translateX(-4px); }
  80% { transform: translateX(4px); }
}
@keyframes shimmer {
  0%, 70% { left: -40%; }
  100% { left: 140%; }
}
@keyframes flap {
  from { transform: scaleY(1); }
  to { transform: scaleY(0.55); }
}
.animate-glow {
  animation: glow 1.6s ease-in-out infinite;
}
.shake {
  animation: shake 0.45s ease-in-out;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.35s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
.swap-enter-active,
.swap-leave-active {
  transition: opacity 0.2s ease, transform 0.2s ease;
}
.swap-enter-from {
  opacity: 0;
  transform: translateY(6px);
}
.swap-leave-to {
  opacity: 0;
  transform: translateY(-6px);
}

/* ================= Scrollbar ================= */
.custom-scrollbar {
  scrollbar-width: thin;
  scrollbar-color: #ec4899 transparent;
}
.custom-scrollbar::-webkit-scrollbar {
  width: 8px;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background: #ec4899;
  border-radius: 9999px;
}

/* ================= Aksesibilitas ================= */
@media (prefers-reduced-motion: reduce) {
  .mascot,
  .animate-glow,
  .shake,
  .spinner,
  .submit::after,
  .submit__bat {
    animation: none !important;
  }
  .toggle__pill,
  .submit,
  .drop,
  .loading__fill {
    transition: none;
  }
}
</style>