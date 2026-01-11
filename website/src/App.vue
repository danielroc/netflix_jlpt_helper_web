<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { translations } from './locales'
import screenshotEn from './assets/screenshot-en.png'
import screenshotJa from './assets/screenshot-ja.png'
import screenshotZh from './assets/screenshot-zh.png'
import flashcardScreenshot from './assets/flashcard-r.png'
import movie from './assets/movie.mov'

const detectLanguage = () => {
  const navLang = navigator.language || navigator.userLanguage
  const supported = ['zh-TW', 'en', 'ja', 'ko', 'vi', 'th', 'id']
  
  // Try exact match
  if (supported.includes(navLang)) return navLang
  
  // Try prefix match (e.g., 'en-US' -> 'en')
  const shortLang = navLang.split('-')[0]
  if (shortLang === 'zh') return 'zh-TW' // Default zh to zh-TW
  if (supported.includes(shortLang)) return shortLang
  
  return 'en' // Fallback
}

const currentLang = ref(detectLanguage())

const t = (key) => {
  return translations[currentLang.value][key] || translations['en'][key] || key
}

const currentScreenshot = computed(() => {
  if (currentLang.value === 'zh-TW') return screenshotZh
  if (currentLang.value === 'ja') return screenshotJa
  return screenshotEn
})

// SEO Update logic
watch(currentLang, (newLang) => {
  const title = t('seo_title')
  const desc = t('seo_desc')
  document.title = title
  const metaDesc = document.querySelector('meta[name="description"]')
  if (metaDesc) {
    metaDesc.setAttribute('content', desc)
  }
}, { immediate: true })

const languages = [
  { code: 'zh-TW', name: '繁體中文', flag: '🇹🇼' },
  { code: 'en', name: 'English', flag: '🇺🇸' },
  { code: 'ja', name: '日本語', flag: '🇯🇵' },
  { code: 'ko', name: '한국어', flag: '🇰🇷' },
  { code: 'vi', name: 'Tiếng Việt', flag: '🇻🇳' },
  { code: 'th', name: 'ไทย', flag: '🇹🇭' },
  { code: 'id', name: 'Bahasa Indonesia', flag: '🇮🇩' }
]

const features = computed(() => [
  {
    title: t('feature_jlpt_title'),
    desc: t('feature_jlpt_desc'),
    icon: '📊'
  },
  {
    title: t('feature_trans_title'),
    desc: t('feature_trans_desc'),
    icon: '🔍'
  },
  {
    title: t('feature_token_title'),
    desc: t('feature_token_desc'),
    icon: '🧩'
  },
  {
    title: t('feature_learn_title'),
    desc: t('feature_learn_desc'),
    icon: '📖'
  },
  {
    title: t('feature_flash_title'),
    desc: t('feature_flash_desc'),
    icon: '🎴'
  },
  {
    title: t('feature_exam_title'),
    desc: t('feature_exam_desc'),
    icon: '💯',
    isComingSoon: true
  }
])
const mockSubtitles = [
  { text: '迅速', level: 'n1', reading: 'じんそく', trans: 'Rapid, quick', isMain: true, highlight: true  },
  { text: 'な', level: 'n5', reading: '', trans: 'Copula (modifying)' },
  { text: '発展', level: 'n3', reading: 'はってん', trans: 'Development' },
  { text: 'は', level: 'n5', reading: '', trans: 'Topic marker' },
  { text: '私', level: 'n5', reading: 'わたし', trans: 'I, me' },
  { text: 'の', level: 'n5', reading: '', trans: 'Possessive particle' },
  { text: '日常', level: 'n2', reading: 'にちじょう', trans: 'Daily life' },
  { text: 'を', level: 'n5', reading: '', trans: 'Object marker' },
  { text: '変え', level: 'n4', reading: 'かえ', trans: 'To change' },
  { text: 'ました', level: 'n5', reading: '', trans: 'Polite past suffix' }
]

const hoveredWord = ref(null)
const mousePos = ref({ x: 0, y: 0 })
const tooltipPos = ref({ x: 0, y: 0 })
const phantomPos = ref({ x: 0, y: 0 })
const isAutoPlaying = ref(true)
let autoPlayInterval = null
let closeTimeout = null
const wordRefs = ref([])
const playerScreenRef = ref(null)

// Testimonial Carousel
const testimonialIndex = ref(0)
const testimonials = computed(() => [
  { user: t('t1_user'), text: t('t1_text'), icon: '🧑‍🎓' },
  { user: t('t2_user'), text: t('t2_text'), icon: '👨‍💻' },
  { user: t('t3_user'), text: t('t3_text'), icon: '👩‍🏫' }
])

onMounted(() => {
  setInterval(() => {
    testimonialIndex.value = (testimonialIndex.value + 1) % testimonials.value.length
  }, 5000)
})

const startAutoPlay = () => {
  let index = 0
  autoPlayInterval = setInterval(() => {
    if (isAutoPlaying.value) {
      const word = mockSubtitles[index]
      hoveredWord.value = word
      
      // Calculate position for phantom cursor and tooltip relative to player screen
      const el = wordRefs.value[index]
      const playerEl = playerScreenRef.value
      if (el && playerEl) {
        const rect = el.getBoundingClientRect()
        const playerRect = playerEl.getBoundingClientRect()
        
        phantomPos.value = {
          x: rect.left - playerRect.left + rect.width / 2,
          y: rect.top - playerRect.top + rect.height / 2 + 10
        }

        tooltipPos.value = {
          x: rect.right - playerRect.left,
          y: rect.top - playerRect.top
        }
      }
      
      index = (index + 1) % mockSubtitles.length
    }
  }, 2000)
}

onMounted(() => {
  // Set default
  hoveredWord.value = mockSubtitles.find(s => s.isMain)
  startAutoPlay()
})

const handleHover = (word, index) => {
  if (closeTimeout) clearTimeout(closeTimeout)
  isAutoPlaying.value = false
  hoveredWord.value = word

  // Calculate top-right position of the word
  const el = wordRefs.value[index]
  const playerEl = playerScreenRef.value
  if (el && playerEl) {
    const rect = el.getBoundingClientRect()
    const playerRect = playerEl.getBoundingClientRect()
    tooltipPos.value = {
      x: rect.right - playerRect.left,
      y: rect.top - playerRect.top
    }
  }
}

const handleMouseMove = (event) => {
  if (hoveredWord.value) {
    mousePos.value = { x: event.clientX, y: event.clientY }
  }
}

const handleMouseLeave = () => {
  if (closeTimeout) clearTimeout(closeTimeout)
  closeTimeout = setTimeout(() => {
    hoveredWord.value = mockSubtitles.find(s => s.isMain)
    isAutoPlaying.value = true
  }, 300)
}

const keepTooltip = () => {
  if (closeTimeout) clearTimeout(closeTimeout)
}

const isMenuOpen = ref(false)

const toggleMenu = () => {
  isMenuOpen.value = !isMenuOpen.value
}

const trackEvent = (eventName, params = {}) => {
  if (typeof window.gtag === 'function') {
    window.gtag('event', eventName, params)
  }
}
</script>


<template>
  <div class="container" :class="[`lang-${currentLang}`, { 'menu-open': isMenuOpen }]">
    <nav class="pixel-nav">
      <div class="logo-box">
        <img src="./assets/logo.png" alt="NihonGo!" class="logo-img" />
        <span>NihonGo!</span>
      </div>

      <button class="burger-btn pixel-border" @click="toggleMenu">
        <span></span>
        <span></span>
        <span></span>
      </button>

      <div class="nav-links" :class="{ 'active': isMenuOpen }">
        <a href="#demo" @click="isMenuOpen = false; trackEvent('view_demo')">{{ t('section_demo') }}</a>
        <a href="#features" @click="isMenuOpen = false">{{ t('nav_features') }}</a>
        
        <div class="lang-switcher">
          <select v-model="currentLang" class="lang-select pixel-border" @change="isMenuOpen = false">
            <option v-for="lang in languages" :key="lang.code" :value="lang.code">
              {{ lang.flag }} {{ lang.name }}
            </option>
          </select>
        </div>

        <a href="https://ko-fi.com/codeplay0301" target="_blank" class="pixel-btn sm coffee-btn-red" @click="trackEvent('click_donate')">
          <span class="coffee-icon">☕️</span> {{ t('nav_donate') }}
        </a>
      </div>
    </nav>

    <div class="pixel-stars"></div>

    <header class="hero section">
      <div class="hero-content">
        <h1 class="hero-title">{{ t('hero_title') }}</h1>
        <p class="hero-subtitle">{{ t('hero_subtitle') }}</p>
        <p class="hero-description">{{ t('hero_description') }}</p>
        <div class="hero-stats-box pixel-border-sm">
          <span class="stats-icon">📈</span>
          {{ t('hero_stats') }}
        </div>
        
        <div class="global-support-badge pixel-border-sm">
           <span class="support-label">Supported in 7 Languages</span>
           <div class="support-flags">🇹🇼 🇺🇸 🇯🇵 🇰🇷 🇻🇳 🇹🇭 🇮🇩</div>
        </div>

        <div class="hero-actions">
          <a href="#" class="pixel-btn lg" @click="trackEvent('click_install')">{{ t('btn_install') }}</a>
          <a href="#demo" class="pixel-btn secondary lg" @click="trackEvent('view_demo')">{{ t('btn_demo') }}</a>
        </div>
      </div>
      <div class="hero-visual">
        <div class="landscape-frame pixel-border">
          <video :src="movie" autoplay loop muted playsinline class="hero-img-landscape"></video>
          <div class="scanlines"></div>
        </div>
      </div>
    </header>

    <section id="demo" class="demo section">
      <h2 class="section-title">{{ t('section_demo') }}</h2>

      <!-- Real Screenshot Display -->
      <div class="mock-player pixel-border" style="margin-bottom: 3rem; cursor: default;">
        <div class="player-header">
          <div class="dot red"></div>
          <div class="dot yellow"></div>
          <div class="dot green"></div>
          <span class="player-title">{{ t('player_title') }}</span>
        </div>
        <div class="player-screen-static">
          <img src="./assets/screenshot-r.png" alt="Extension Demo" class="demo-screenshot" />
          <div class="scanlines"></div>
        </div>
      </div>

      <!-- Interactive Mock Demo -->
      <div class="mock-player pixel-border" @mousemove="handleMouseMove" @mouseleave="handleMouseLeave">
        <div class="player-header">
          <div class="dot red"></div>
          <div class="dot yellow"></div>
          <div class="dot green"></div>
          <span class="player-title">{{ t('player_title') }}</span>
        </div>
        <div class="player-screen" ref="playerScreenRef">
          <div class="video-bg">
            <div class="pixel-city"></div>
            <div class="city-lights"></div>
          </div>
          <div class="scanlines"></div>
          <div class="subtitle-box">
            <span 
              v-for="(word, index) in mockSubtitles" 
              :key="word.text"
              :ref="el => { if (el) wordRefs[index] = el }"
              class="word"
              :class="[word.level, { active: hoveredWord === word, highlight: word.highlight }]"
              @mouseover="handleHover(word, index)"
              @mouseleave="handleMouseLeave"
            >
              {{ word.text }}
            </span>
          </div>
          
          <transition name="fade">
            <div 
              v-if="hoveredWord" 
              class="tooltip pixel-border"
              :class="{ 'fixed-tip': true }"
              :style="{ left: tooltipPos.x + 'px', top: tooltipPos.y + 'px' }"
              @mouseover="keepTooltip"
              @mouseleave="handleMouseLeave"
            >
              <div class="tooltip-header">
                {{ hoveredWord.text }} {{ hoveredWord.reading ? `(${hoveredWord.reading})` : '' }}
              </div>
              <div class="tooltip-body">
                <span class="tag" :class="hoveredWord.level">
                  {{ hoveredWord.level.toUpperCase() }}
                </span>
                <p>{{ hoveredWord.trans }}</p>
                <div class="tooltip-actions">
                  <button class="action-btn" title="Copy">
                    <span class="btn-icon">📋</span>
                    <span class="btn-text">Copy</span>
                  </button>
                  <button class="action-btn" title="Add to Flashcards">
                    <span class="btn-icon">🎴</span>
                    <span class="btn-text">Add</span>
                  </button>
                </div>
              </div>
            </div>
          </transition>

          <!-- Real Cursor Label -->
          <div 
            v-if="hoveredWord && !isAutoPlaying" 
            class="level-cursor pixel-border"
            :class="hoveredWord.level"
            :style="{ left: mousePos.x + 'px', top: mousePos.y + 'px' }"
          >
            {{ hoveredWord.level.toUpperCase() }}
          </div>

          <!-- Phantom Cursor for Auto-play -->
          <div 
            v-if="isAutoPlaying" 
            class="phantom-cursor"
            :style="{ left: phantomPos.x + 'px', top: phantomPos.y + 'px' }"
          >
            👆
          </div>
        </div>
      </div>
    </section>

    <section id="features" class="features section">
      <h2 class="section-title">{{ t('section_features') }}</h2>
      <div class="feature-grid">
        <div v-for="f in features" :key="f.title" class="feature-card pixel-border">
          <div class="feature-icon">{{ f.icon }}</div>
          <h3 class="feature-title">{{ f.title }}</h3>
          <p class="feature-desc">{{ f.desc }}</p>
          <div v-if="f.isComingSoon" class="coming-soon-badge">
            <span class="blink">✨</span> {{ t('label_coming_soon') }}
          </div>
        </div>
      </div>
    </section>

    <section class="flashcard-showcase section">
      <div class="simplicity-container reverse">
        <div class="simplicity-content">
          <h2 class="section-title simplicity-title">{{ t('flashcard_section_title') }}</h2>
          <h3 class="simplicity-subtitle">{{ t('flashcard_section_subtitle') }}</h3>
          <p class="simplicity-desc">{{ t('flashcard_section_desc') }}</p>
        </div>
        <div class="simplicity-visual">
          <div class="browser-frame pixel-border">
             <div class="browser-header">
              <div class="dot red"></div>
              <div class="dot yellow"></div>
              <div class="dot green"></div>
              <div class="browser-url">nihongo.app/flashcards</div>
            </div>
            <img :src="flashcardScreenshot" alt="Flashcards" class="popup-screenshot" />
          </div>
        </div>
      </div>
    </section>



    <section class="simplicity section">
      <div class="simplicity-container">
        <div class="simplicity-content">
          <h2 class="section-title simplicity-title">{{ t('simplicity_title') }}</h2>
          <h3 class="simplicity-subtitle">{{ t('simplicity_subtitle') }}</h3>
          <p class="simplicity-desc">{{ t('simplicity_desc') }}</p>
        </div>
        <div class="simplicity-visual">
          <div class="browser-frame pixel-border">
            <div class="browser-header">
              <div class="dot red"></div>
              <div class="dot yellow"></div>
              <div class="dot green"></div>
              <div class="browser-url">nihongo.app</div>
            </div>
            <img :src="currentScreenshot" alt="Extension Popup" class="popup-screenshot" />
          </div>
        </div>
      </div>
    </section>

    <section class="testimonials section">
      <h2 class="section-title">{{ t('section_testimonials') }}</h2>
      <div class="testimonial-carousel pixel-border">
        <transition name="carousel" mode="out-in">
          <div :key="testimonialIndex" class="testimonial-card">
            <div class="testimonial-icon">{{ testimonials[testimonialIndex].icon }}</div>
            <p class="testimonial-text">"{{ testimonials[testimonialIndex].text }}"</p>
            <span class="testimonial-user">- {{ testimonials[testimonialIndex].user }}</span>
          </div>
        </transition>
        <div class="carousel-dots">
          <span 
            v-for="(_, i) in testimonials" 
            :key="i"
            class="dot"
            :class="{ active: i === testimonialIndex }"
            @click="testimonialIndex = i"
          ></span>
        </div>
      </div>
    </section>

    <footer class="footer">
      <p class="footer-made">{{ t('footer_made') }}</p>
      <p class="footer-cta">{{ t('footer_cta') }}</p>
      
      <div class="footer-actions">
        <a href="mailto:service@codeplay.tw" class="footer-contact-btn pixel-border-sm">
          <span class="contact-icon">📧</span> service@codeplay.tw
        </a>
        <a href="https://ko-fi.com/codeplay0301" target="_blank" class="pixel-btn sm coffee-btn-red" @click="trackEvent('click_donate')">
          <span class="coffee-icon">☕️</span> Buy Me a Coffee
        </a>
      </div>

      <div class="copyright">{{ t('footer_copyright') }}</div>
    </footer>
  </div>
</template>

<style scoped>
.container {
  max-width: 1400px;
  margin: 0 auto;
}

.pixel-nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 2rem;
  background: var(--secondary-color);
  margin-top: 1rem;
}

.logo-box {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.logo-img {
  width: 48px;
  height: 48px;
}

.nav-links {
  display: flex;
  align-items: center;
  gap: 1.5rem;
}

.nav-links a {
  color: white;
  text-decoration: none;
  font-size: 1rem;
}

.lang-select {
  background: var(--bg-color);
  color: white;
  border: 2px solid white;
  padding: 0 0.8rem;
  font-family: inherit;
  font-size: 0.9rem;
  line-height: normal;
  cursor: pointer;
  outline: none;
  height: 3rem;
  width: 160px;
}

.lang-select option {
  background: var(--secondary-color);
}

.lang-ko { font-family: 'Galmuri11', 'Galmuri9', sans-serif; }
.lang-th { font-family: 'Chakra Petch', sans-serif; font-weight: 700; }
.lang-vi, .lang-id { font-family: 'Pixelify Sans', sans-serif; }

/* English Font Size Optimization */
.lang-en .hero-title { font-size: 2.2rem; }
.lang-en .hero-subtitle { font-size: 0.9rem; }
.lang-en .hero-description { font-size: 0.8rem; letter-spacing: 0.2px; }
.lang-en .hero-stats-box { font-size: 0.7rem; padding: 0.8rem 1.2rem; }
.lang-en .nav-links a { font-size: 0.8rem; }
.lang-en .pixel-btn.lg { font-size: 0.8rem; padding: 1rem 2rem; }
.lang-en .pixel-btn.sm { font-size: 0.7rem; }
.lang-en .coffee-btn-red { font-size: 0.75rem !important; padding: 0.5rem 1rem !important; }
.lang-en .section-title { font-size: 1.5rem; }
.lang-en .feature-title { font-size: 0.8rem; }
.lang-en .feature-desc { font-size: 0.65rem; }
.lang-en .testimonial-text { font-size: 0.9rem; }
.lang-en .footer { font-size: 0.65rem; }
.lang-en .footer-cta { font-size: 0.7rem; }
.lang-en .coffee-link-yellow { font-size: 0.75rem; }

.section {
  padding: 4rem 2rem;
}

.hero {
  display: flex;
  flex-direction: row-reverse;
  align-items: center;
  justify-content: space-between;
  gap: 3rem;
  min-height: 50vh;
}

.hero-content {
  flex: 1;
}

.hero-visual {
  flex: 1.5;
}

.landscape-frame {
  position: relative;
  overflow: hidden;
  box-shadow: 20px 20px 0 var(--primary-color);
  background: #000;
}

.hero-img-landscape {
  width: 100%;
  display: block;
  image-rendering: pixelated;
}

.pixel-stars {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: -1;
  background: radial-gradient(1px 1px at 20px 30px, #fff, rgba(0,0,0,0)),
              radial-gradient(1px 1px at 40px 70px, #fff, rgba(0,0,0,0)),
              radial-gradient(1px 1px at 50px 160px, #fff, rgba(0,0,0,0)),
              radial-gradient(2px 2px at 90px 40px, #fff, rgba(0,0,0,0)),
              radial-gradient(2px 2px at 130px 80px, #fff, rgba(0,0,0,0)),
              radial-gradient(2px 2px at 160px 120px, #fff, rgba(0,0,0,0));
  background-repeat: repeat;
  background-size: 200px 200px;
  animation: starsTwinkle 4s infinite linear;
}

@keyframes starsTwinkle {
  0% { opacity: 0.5; }
  50% { opacity: 1; }
  100% { opacity: 0.5; }
}

.hero-title {
  font-size: 3.5rem;
  margin-bottom: 2rem;
  text-shadow: 4px 4px 0px var(--primary-color);
  line-height: 1.1;
}

.hero-subtitle {
  font-size: 1.2rem;
  margin-bottom: 1rem;
  color: var(--primary-color);
  font-weight: bold;
}

.hero-description {
  font-size: 1rem;
  margin-bottom: 2rem;
  color: #ccc;
  line-height: 1.6;
}

.hero-stats-box {
  background: rgba(229, 9, 20, 0.1);
  border-color: var(--primary-color);
  padding: 1rem 1.5rem;
  margin-bottom: 3rem;
  font-size: 0.9rem;
  color: #fff;
  display: flex;
  align-items: center;
  gap: 15px;
  max-width: fit-content;
}

.global-support-badge {
  background: rgba(255, 255, 255, 0.1);
  border-color: #bdc3c7;
  padding: 0.8rem 1.2rem;
  margin-bottom: 2rem;
  display: inline-block;
}
.support-label {
  display: block;
  font-size: 0.7rem;
  color: #bdc3c7;
  margin-bottom: 0.5rem;
  text-transform: uppercase;
  letter-spacing: 1px;
}
.support-flags {
  font-size: 1.2rem;
  letter-spacing: 5px;
}

.stats-icon {
  font-size: 1.5rem;
}

.hero-actions {
  display: flex;
  gap: 2rem;
}

.pixel-btn.lg {
  padding: 1.5rem 2.5rem;
  font-size: 1rem;
}

.pixel-btn.sm {
  padding: 0.5rem 1rem;
  font-size: 0.6rem;
}

.pixel-btn.secondary {
  background-color: #576574;
  box-shadow: inset -4px -4px 0 0 #2f3542;
}
.coffee-btn-red {
  background-color: var(--primary-color) !important;
  color: #ffffff !important;
  box-shadow: inset -4px -4px 0 0 #500000 !important;
  font-size: 0.9rem !important;
  padding: 0.6rem 1.2rem !important;
  display: flex !important;
  align-items: center !important;
  gap: 8px !important;
}
.coffee-icon {
  font-size: 1.4rem;
  line-height: 1;
}
.footer-links {
  margin: 10px 0;
}

.section-title {
  text-align: center;
  margin-bottom: 2rem;
  font-size: 2rem;
}

.feature-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 3rem;
}

.simplicity-container {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 4rem;
}

.simplicity-container.reverse {
  flex-direction: row-reverse;
}

@media (max-width: 768px) {
  .simplicity-container, .simplicity-container.reverse {
    flex-direction: column-reverse;
    text-align: center;
  }
}

.simplicity-content {
  flex: 1;
  text-align: left;
}

.simplicity-visual {
  flex: 1.2;
  display: flex;
  justify-content: center;
}

.simplicity .simplicity-visual .browser-frame {
  max-width: 380px;
}

.simplicity-title {
  text-align: left;
  margin-bottom: 1.5rem;
  line-height: 1.3;
}

.simplicity-subtitle {
  font-size: 1.2rem;
  color: var(--primary-color);
  margin-bottom: 2rem;
  line-height: 1.5;
}

.simplicity-desc {
  font-size: 1rem;
  color: #ccc;
  line-height: 1.8;
}

.browser-frame {
  background: #1e272e;
  border-radius: 4px;
  overflow: hidden;
  max-width: 600px;
  width: 100%;
  box-shadow: 15px 15px 0 rgba(0,0,0,0.3);
}

.browser-header {
  background: #2f3542;
  padding: 0.5rem 1rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.browser-url {
  background: #1e272e;
  color: #7f8c8d;
  font-size: 0.6rem;
  padding: 2px 8px;
  border-radius: 2px;
  margin-left: 0.5rem;
  flex: 1;
  text-align: center;
}

.popup-screenshot {
  width: 100%;
  display: block;
}

.feature-card {
  position: relative;
  padding: 2rem;
  background: #2f3542;
  text-align: center;
  transition: transform 0.2s;
}

.feature-card:hover {
  transform: scale(1.05);
}

.coming-soon-badge {
  position: absolute;
  top: -10px;
  right: -10px;
  background: #ff4757;
  color: white;
  font-size: 0.6rem;
  padding: 4px 8px;
  border: 2px solid white;
  box-shadow: 2px 2px 0 rgba(0,0,0,0.5);
  transform: rotate(5deg);
  z-index: 10;
  font-weight: bold;
}

.blink {
  animation: blink 1s infinite;
}

@keyframes blink {
  0% { opacity: 1; }
  50% { opacity: 0; }
  100% { opacity: 1; }
}

.feature-icon {
  font-size: 3rem;
  margin-bottom: 1.5rem;
}

.feature-title {
  margin-bottom: 1rem;
  font-size: 1rem;
}

.feature-desc {
  font-size: 0.7rem;
  color: #bdc3c7;
  line-height: 1.8;
}

.mock-player {
  background: #000;
  overflow: hidden;
  cursor: crosshair;
}

.player-header {
  background: #333;
  padding: 0.5rem 1rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
  position: relative;
  z-index: 10;
}

.dot {
  width: 12px;
  height: 12px;
  border-radius: 50%;
}

.red { background: #ff5f56; }
.yellow { background: #ffbd2e; }
.green { background: #27c93f; }

.player-title {
  font-size: 0.6rem;
  margin-left: 1rem;
  color: #999;
}

.player-screen {
  height: 400px;
  position: relative;
  display: flex;
  flex-direction: column;
  justify-content: flex-end;
  align-items: center;
  padding-bottom: 4rem;
  background: #000;
  overflow: hidden;
  box-shadow: inset 0 0 100px rgba(0,0,0,0.8);
}

.player-screen-static {
  position: relative;
  width: 100%;
  background: #000;
  overflow: hidden;
  line-height: 0;
}

.demo-screenshot {
  width: 100%;
  height: auto;
  display: block;
}

.video-bg {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 0;
}

.pixel-city {
  width: 100%;
  height: 100%;
  background: linear-gradient(180deg, #1a1a2e 0%, #16213e 100%);
  position: relative;
}

.city-lights {
  position: absolute;
  bottom: 0;
  width: 100%;
  height: 60%;
  background: repeating-linear-gradient(
    90deg,
    transparent 0px,
    transparent 40px,
    rgba(255, 235, 153, 0.1) 40px,
    rgba(255, 235, 153, 0.1) 80px
  );
  animation: bgMove 20s linear infinite;
}

@keyframes bgMove {
  from { background-position: 0 0; }
  to { background-position: 400px 0; }
}

.scanlines {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(
    rgba(18, 16, 16, 0) 50%,
    rgba(0, 0, 0, 0.1) 50%
  ),
  linear-gradient(
    90deg,
    rgba(255, 0, 0, 0.03),
    rgba(0, 255, 0, 0.01),
    rgba(0, 0, 255, 0.03)
  );
  background-size: 100% 4px, 100% 100%;
  z-index: 1;
  pointer-events: none;
  animation: flicker 0.15s infinite;
}

@keyframes flicker {
  0% { opacity: 0.9; }
  100% { opacity: 1; }
}

.subtitle-box {
  display: flex;
  gap: 0.5rem;
  font-size: 2rem;
  z-index: 2;
  transition: transform 0.3s;
}

.word {
  cursor: none;
  padding: 0.2rem;
  border-bottom: 4px solid transparent;
}

.word.active {
  background: rgba(255,255,255,0.2);
}

.n1 { border-color: #ff4757; }
.n2 { border-color: #ffa502; }
.n3 { border-color: #2ed573; }
.n4 { border-color: #1e90ff; }

.highlight {
  background: var(--primary-color);
  color: white;
  border-color: white;
}

.level-cursor {
  position: fixed;
  pointer-events: none;
  transform: translate(15px, 15px);
  padding: 4px 8px;
  font-size: 0.7rem;
  color: white;
  z-index: 9999;
  background: #000;
}

.level-cursor.n1 { color: #ff4757; border-color: #ff4757; }
.level-cursor.n2 { color: #ffa502; border-color: #ffa502; }
.level-cursor.n3 { color: #2ed573; border-color: #2ed573; }
.level-cursor.n4 { color: #1e90ff; border-color: #1e90ff; }

.tooltip {
  background: var(--secondary-color);
  padding: 1rem;
  min-width: 250px;
  z-index: 10000;
  box-shadow: 10px 10px 0 rgba(0,0,0,0.5);
  pointer-events: auto; /* Enable interaction */
}

.fixed-tip {
  position: absolute; /* Change to absolute relative to player-screen */
  transform: translate(10px, -110%); /* Shift right and up */
}

.auto-tip {
  position: absolute;
  top: 10%;
  left: 50%;
  transform: translateX(-50%);
  animation: slideUp 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}

.phantom-cursor {
  position: absolute;
  pointer-events: none;
  font-size: 2rem;
  z-index: 9998;
  transform: translate(-50%, -10px);
  transition: all 0.5s cubic-bezier(0.165, 0.84, 0.44, 1);
  filter: drop-shadow(2px 2px 0 rgba(0,0,0,0.5));
}

@keyframes slideUp {
  from { opacity: 0; transform: translate(-50%, 20px); }
  to { opacity: 1; transform: translate(-50%, 0); }
}

.tooltip-header {
  font-size: 1.1rem;
  margin-bottom: 0.8rem;
  border-bottom: 2px solid #555;
  padding-bottom: 0.5rem;
}

.tooltip-body {
  font-size: 0.8rem;
}

.tooltip-actions {
  display: flex;
  gap: 0.8rem;
  margin-top: 1.2rem;
}

.action-btn {
  background: #3d4451;
  border: 2px solid #666;
  color: white;
  padding: 6px 10px;
  cursor: pointer;
  font-size: 0.8rem;
  display: flex;
  align-items: center;
  gap: 6px;
  transition: all 0.1s;
  box-shadow: 4px 4px 0px rgba(0,0,0,0.3);
}

.action-btn:hover {
  background: #57606f;
  border-color: white;
  transform: translate(-1px, -1px);
  box-shadow: 5px 5px 0px rgba(0,0,0,0.4);
}

.action-btn:active {
  transform: translate(2px, 2px);
  box-shadow: 1px 1px 0px rgba(0,0,0,0.4);
}

.btn-icon {
  font-size: 1.1rem;
}

.btn-text {
  font-size: 0.7rem;
  text-transform: uppercase;
  font-weight: bold;
}

.tag {
  display: inline-block;
  padding: 0.2rem 0.6rem;
  font-size: 0.6rem;
  margin-bottom: 0.8rem;
  color: white;
}

.tag.n1 { background: #ff4757; }
.tag.n2 { background: #ffa502; }
.tag.n3 { background: #2ed573; }
.tag.n3 { background: #2ed573; }
.tag.n4 { background: #1e90ff; }

/* Testimonials Styles */
.testimonial-carousel {
  max-width: 800px;
  margin: 0 auto;
  background: #1e272e;
  padding: 3rem;
  text-align: center;
  min-height: 250px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  position: relative;
  box-shadow: 10px 10px 0 var(--secondary-color);
}

.testimonial-icon {
  font-size: 3rem;
  margin-bottom: 1rem;
}

.testimonial-text {
  font-size: 1.2rem;
  line-height: 1.6;
  margin-bottom: 1.5rem;
  font-style: italic;
  color: #fff;
}

.testimonial-user {
  font-size: 0.9rem;
  color: #aaa;
  font-weight: bold;
}

.carousel-dots {
  display: flex;
  justify-content: center;
  gap: 1rem;
  margin-top: 2rem;
}

.carousel-dots .dot {
  width: 10px;
  height: 10px;
  background: #555;
  cursor: pointer;
}

.carousel-dots .dot.active {
  background: var(--primary-color);
  box-shadow: 0 0 10px var(--primary-color);
}

.carousel-enter-active, .carousel-leave-active {
  transition: all 0.5s ease;
}

.carousel-enter-from { opacity: 0; transform: translateX(20px); }
.carousel-leave-to { opacity: 0; transform: translateX(-20px); }

.footer {
  text-align: center;
  padding: 4rem 2rem 2rem;
  font-size: 0.75rem;
  color: #7f8c8d;
  background: #151515;
  border-top: 2px solid #333;
  margin-top: 4rem;
}

.footer-made {
  color: #aaa;
  margin-bottom: 0.5rem;
  font-size: 0.9rem;
}

.footer-cta {
  color: #fff;
  margin-bottom: 2rem;
  font-size: 1rem;
  font-weight: bold;
}

.footer-actions {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1.5rem;
  margin-bottom: 3rem;
  flex-wrap: wrap;
}

.footer-contact-btn {
  display: flex;
  align-items: center;
  gap: 8px;
  background: #2f3542;
  color: #fff;
  padding: 0.6rem 1.2rem;
  text-decoration: none;
  font-size: 0.9rem;
  transition: all 0.2s;
}

.footer-contact-btn:hover {
  background: #57606f;
  transform: translateY(-2px);
}

.contact-icon {
  font-size: 1.2rem;
}

.copyright {
  color: #555;
  font-size: 0.7rem;
}
.floating {
  animation: floating 3s ease-in-out infinite;
  width: 300px;
}

@keyframes floating {
  0% { transform: translateY(0px); }
  50% { transform: translateY(-20px); }
  100% { transform: translateY(0px); }
}

.burger-btn {
  display: none;
  flex-direction: column;
  justify-content: space-around;
  width: 40px;
  height: 40px;
  background: var(--bg-color);
  padding: 8px;
  cursor: pointer;
  z-index: 1001;
}

.burger-btn span {
  width: 100%;
  height: 4px;
  background: white;
}

@media (max-width: 768px) {
  .burger-btn {
    display: flex;
  }

  .pixel-nav {
    flex-direction: row;
    justify-content: space-between;
    padding: 1rem;
    position: relative;
    z-index: 1000;
  }

  .nav-links {
    display: none;
    position: absolute;
    top: 100%;
    left: 0;
    right: 0;
    background: var(--secondary-color);
    flex-direction: column;
    padding: 2rem;
    gap: 1.5rem;
    border-bottom: 4px solid white;
  }

  .nav-links.active {
    display: flex;
  }

  .lang-switcher {
    width: auto;
  }

  .lang-select {
    width: auto;
    min-width: 160px;
    text-align: center;
    text-align-last: center;
  }

  .lang-select {
    height: 3.5rem;
  }
  .hero {
    flex-direction: column-reverse;
    text-align: center;
    padding: 2rem 1rem;
    gap: 2rem;
    min-height: auto;
  }
  .hero-actions {
    flex-direction: column;
    width: 100%;
    gap: 1rem;
  }
  .hero-actions .pixel-btn {
    width: 100%;
    text-align: center;
  }
  .hero-image {
    display: none;
  }
  .hero-title {
    font-size: 2rem;
    line-height: 1.2;
  }
  .hero-subtitle {
    font-size: 0.9rem;
  }

  /* English-specific mobile adjustments */
  .lang-en .hero-title { font-size: 1.6rem; }
  .lang-en .hero-subtitle { font-size: 0.75rem; }
  .lang-en .hero-description { font-size: 0.7rem; }
  .lang-en .hero-stats-box { font-size: 0.65rem; }
  .lang-en .section-title { font-size: 1.2rem; }
  .section-title {
    font-size: 1.5rem;
    margin-bottom: 2rem;
  }
  .player-screen {
    height: auto;
    min-height: 400px;
    padding: 0;
    display: flex;
    flex-direction: column;
    justify-content: flex-start;
  }
  .subtitle-box {
    order: 1;
    font-size: 1.2rem;
    flex-wrap: wrap;
    justify-content: center;
    padding: 3rem 1rem;
    gap: 0.5rem;
    background: #000;
    width: 100%;
  }
  .tooltip {
    order: 2;
    position: relative !important;
    top: 0 !important;
    left: 0 !important;
    transform: none !important;
    width: 100% !important;
    min-width: auto;
    z-index: 10;
    background: #222 !important; /* Separate from pure black screen */
    border: none;
    border-top: 4px solid #444;
    box-shadow: none !important;
    margin: 0;
    flex-grow: 1;
    animation: none !important;
  }
  .fixed-tip {
    position: relative !important;
    transform: none !important;
  }
  .phantom-cursor {
    font-size: 1.5rem;
  }
  .level-cursor {
    display: none;
  }
  .feature-grid {
    gap: 1.5rem;
    grid-template-columns: 1fr;
  }
  .feature-card {
    padding: 1.5rem;
  }

  /* Simplicity Mobile */
  .simplicity-container {
    flex-direction: column;
    gap: 3rem;
  }
  .simplicity-content {
    text-align: center;
  }
  .simplicity-title {
    text-align: center;
    font-size: 1.5rem;
  }
  .simplicity-visual {
    width: 100%;
  }
}
</style>
