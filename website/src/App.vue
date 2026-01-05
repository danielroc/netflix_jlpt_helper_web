<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { translations } from './locales'

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
  { code: 'zh-TW', name: '繁體中文' },
  { code: 'en', name: 'English' },
  { code: 'ja', name: '日本語' },
  { code: 'ko', name: '한국어' },
  { code: 'vi', name: 'Tiếng Việt' },
  { code: 'th', name: 'ไทย' },
  { code: 'id', name: 'Bahasa Indonesia' }
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
  }
])
const mockSubtitles = [
  { text: '私', level: 'n1', reading: 'わたし', trans: 'I, me' },
  { text: 'の', level: 'n3', reading: '', trans: 'Possessive particle' },
  { text: '夢', level: 'n2', reading: 'ゆめ', trans: 'Dream' },
  { text: 'は', level: 'n3', reading: '', trans: 'Topic marker' },
  { text: '日本', level: 'n5', reading: 'にほん', trans: 'Japan', isMain: true, highlight: true },
  { text: 'に', level: 'n3', reading: '', trans: 'To, at' },
  { text: '行く', level: 'n4', reading: 'いく', trans: 'To go' },
  { text: 'こと', level: 'n3', reading: '', trans: 'Thing, matter' },
  { text: 'です', level: 'n3', reading: '', trans: 'Be, is' }
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
        <a href="#demo" @click="isMenuOpen = false">{{ t('section_demo') }}</a>
        <a href="#features" @click="isMenuOpen = false">{{ t('nav_features') }}</a>
        
        <div class="lang-switcher">
          <select v-model="currentLang" class="lang-select pixel-border" @change="isMenuOpen = false">
            <option v-for="lang in languages" :key="lang.code" :value="lang.code">
              {{ lang.name }}
            </option>
          </select>
        </div>

        <a href="https://github.com/danielroc0108/netflix_jlpt_helper" target="_blank" class="pixel-btn sm">{{ t('nav_github') }}</a>
      </div>
    </nav>

    <div class="pixel-stars"></div>

    <header class="hero section">
      <div class="hero-content">
        <h1 class="hero-title">{{ t('hero_title') }}</h1>
        <p class="hero-subtitle">{{ t('hero_subtitle') }}</p>
        <div class="hero-actions">
          <a href="#" class="pixel-btn lg">{{ t('btn_install') }}</a>
          <a href="#demo" class="pixel-btn secondary lg">{{ t('btn_demo') }}</a>
        </div>
      </div>
      <div class="hero-visual">
        <div class="landscape-frame pixel-border">
          <img src="/og-image-landscape-s.png" alt="NihonGo! Hero" class="hero-img-landscape" />
          <div class="scanlines"></div>
        </div>
      </div>
    </header>

    <section id="demo" class="demo section">
      <h2 class="section-title">{{ t('section_demo') }}</h2>
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
      <p>{{ t('footer_made') }}</p>
      <div class="copyright">{{ t('footer_copyright') }}</div>
    </footer>
  </div>
</template>

<style scoped>
.container {
  max-width: 1200px;
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

.section {
  padding: 4rem 2rem;
}

.hero {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 3rem;
  min-height: 50vh;
}

.hero-content {
  flex: 1;
}

.hero-visual {
  flex: 1.2;
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
  margin-bottom: 3rem;
  color: #ccc;
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

.feature-card {
  padding: 2rem;
  background: #2f3542;
  text-align: center;
  transition: transform 0.2s;
}

.feature-card:hover {
  transform: scale(1.05);
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
  padding: 3rem 2rem;
  font-size: 0.7rem;
  color: #7f8c8d;
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

  .lang-switcher, .lang-select {
    width: 100%;
    max-width: 100%;
  }

  .lang-select {
    height: 3.5rem;
  }
  .hero {
    flex-direction: column;
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
}
</style>
