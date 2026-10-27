<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const props = defineProps({
  visibleClass: { type: String, default: 'is-visible' },
  selector:     { type: String, default: '[data-reveal]' },
  threshold:    { type: Number, default: 0.15 },
  rootMargin:   { type: String, default: '0px 0px -10% 0px' },
  once:         { type: Boolean, default: true },
})

const root = ref(null)
let observer = null

onMounted(() => {
  if (!('IntersectionObserver' in window)) {
    root.value
      ?.querySelectorAll(props.selector)
      .forEach(el => el.classList.add(props.visibleClass))
    return
  }

  observer = new IntersectionObserver((entries) => {
    for (const entry of entries) {
      if (entry.isIntersecting) {
        entry.target.classList.add(props.visibleClass)
        if (props.once) observer.unobserve(entry.target)
      } else if (!props.once) {
        entry.target.classList.remove(props.visibleClass)
      }
    }
  }, { threshold: props.threshold, rootMargin: props.rootMargin })

  root.value
    ?.querySelectorAll(props.selector)
    .forEach(el => observer.observe(el))
})

onBeforeUnmount(() => observer?.disconnect())
</script>

<template>
  <div ref="root">
    <slot />
  </div>
</template>

<style>
[data-reveal] {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 0.6s ease, transform 0.6s ease;
  will-change: opacity, transform;
}
[data-reveal].is-visible {
  opacity: 1;
  transform: translateY(0);
}

@media (scripting: none) {
  [data-reveal] { opacity: 1 !important; transform: none !important; }
}
</style>