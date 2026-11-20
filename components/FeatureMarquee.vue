<script setup lang="ts">
import { computed, onBeforeUnmount, onMounted, ref } from 'vue'

// Everything cool about Rust that there isn't time to cover.
// Add or remove entries freely - they're shared out across the rows automatically.
const words = [
  'Zero-cost abstractions',
  'Iterators',
  'Closures',
  'Macros',
  'Procedural macros',
  'Lifetimes',
  'async / await',
  'Cargo',
  'Clippy',
  'rustfmt',
  'rustdoc',
  'Built-in testing',
  'crates.io',
  'Workspaces',
  'Type inference',
  'Shadowing',
  'Everything is an expression',
  'Slices',
  'String vs &str',
  'Smart pointers',
  'Box / Rc / Arc',
  'Send & Sync',
  'Const generics',
  'const fn',
  'Monomorphisation',
  'explicit unsafe',
  'Web Assembly',
  'no_std & embedded',
  'Cross-compilation',
  'rust-analyzer',
  'Blanket implementations',
  'Zero-sized types',
  'PhantomData',
  'Typestate pattern',
  'Cow',
  'Pin',
  'No implicit conversions',
  'Integer overflow checks',
  'Tuples',
  'Ranges',
  'Modules & visibility',
  'Conditional compilation',
  'Cargo features',
  'Release profiles',
  'SIMD',
  'Amazing Compiler error messages',
]

const props = withDefaults(defineProps<{
  rows?: number
}>(), {
  rows: 5,
})

// Speed of each row in pixels per second. Rows alternate direction.
const speeds = [35, 60, 22, 48, 30, 55, 26]
// Font size (rem) used for every row.
const size = 1.4

// Each half of a row needs at least this many words so it's always wider than the slide.
const MIN_WORDS_PER_HALF = 12

const lines = computed(() =>
  Array.from({ length: props.rows }, (_, i) => {
    const own = words.filter((_, j) => j % props.rows === i)
    const repeats = Math.max(1, Math.ceil(MIN_WORDS_PER_HALF / Math.max(own.length, 1)))

    return {
      words: Array.from({ length: repeats }, () => own).flat(),
      reverse: i % 2 === 1,
      speed: speeds[i % speeds.length],
    }
  }),
)

// The animation moves each track by half its width, so the duration depends on
// how wide the words actually render. Measure it, and re-measure if it changes
// (e.g. when web fonts finish loading or the slide becomes visible).
const tracks: (HTMLElement | null)[] = []
const durations = ref<number[]>([])
let observer: ResizeObserver | undefined

function setTrack(el: HTMLElement | null, i: number) {
  tracks[i] = el
}

function measure() {
  durations.value = lines.value.map((line, i) => {
    const halfWidth = (tracks[i]?.scrollWidth ?? 0) / 2
    return halfWidth > 0 ? halfWidth / line.speed : 0
  })
}

onMounted(() => {
  measure()
  observer = new ResizeObserver(measure)
  tracks.forEach(track => track && observer!.observe(track))
})

onBeforeUnmount(() => observer?.disconnect())
</script>

<template>
  <div class="feature-marquee" :style="{ fontSize: `${size}rem` }">
    <div
      v-for="(line, i) in lines"
      :key="i"
      class="row"
    >
      <div
        :ref="el => setTrack(el as HTMLElement | null, i)"
        class="track"
        :class="{ reverse: line.reverse, running: durations[i] > 0 }"
        :style="{ animationDuration: `${durations[i] || 0}s` }"
      >
        <!-- the words are rendered twice so the loop is seamless -->
        <span
          v-for="(word, j) in [...line.words, ...line.words]"
          :key="j"
          class="word"
        >{{ word }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
.feature-marquee {
  /* never grow to fit the words - stay as wide as the parent allows */
  min-width: 0;
  contain: inline-size;
  display: flex;
  flex-direction: column;
  gap: 0.6rem;
  overflow: hidden;
  /* fade the words in and out at the edges */
  mask-image: linear-gradient(to right, transparent, black 12%, black 88%, transparent);
  -webkit-mask-image: linear-gradient(to right, transparent, black 12%, black 88%, transparent);
}

.row {
  overflow: hidden;
  white-space: nowrap;
  line-height: 1.3;
  text-align: left;
}

.track {
  display: inline-flex;
  will-change: transform;
}

.track.running {
  animation: scroll linear infinite;
}

.track.reverse {
  animation-direction: reverse;
}

.word {
  padding: 0 0.75em;
  font-weight: 600;
}

.word::after {
  content: '•';
  margin-left: 1.5em;
  opacity: 0.25;
}

@keyframes scroll {
  from {
    transform: translateX(0);
  }
  to {
    transform: translateX(-50%);
  }
}

@media (prefers-reduced-motion: reduce) {
  .track.running {
    animation: none;
  }
}
</style>
