<div align="center">

# ScrollRevealComponent

**Лёгкий Vue 3 компонент для анимации появления элементов при скролле.**  
Без зависимостей. Без магии. Просто работает.

</div>

---

## В чём его плюсы

Стандартный `IntersectionObserver` — мощный инструмент, но каждый раз писать его вручную в новом проекте лень.  
Этот компонент решает одну задачу — **показать элемент, когда он попадает во вьюпорт** — и делает это правильно:

- ✅ Не ломает SSR
- ✅ Не течёт памятью (`unobserve` после появления)
- ✅ Работает с `prefers-reduced-motion`
- ✅ Имеет fallback для браузеров без `IntersectionObserver`
- ✅ Имеет fallback для пользователей без JS (`@media (scripting: none)`)
- ✅ Настраивается через `props`
- ✅ 0 зависимостей

---


### 1. Импортируйте компонент

```vue
<script setup>
import ScrollReveal from '@/components/ScrollReveal.vue'
</script>

<template>
  <ScrollReveal>
    <h2 data-reveal>Обо мне</h2>
    <p data-reveal class="card">Первый блок</p>
    <p data-reveal class="card">Второй блок</p>
  </ScrollReveal>
</template>
```

### 2. Добавьте атрибут `data-reveal` элементам, которые должны появиться

Всё. Компонент сам найдёт их внутри слота и запустит анимацию при попадании в вьюпорт.

---

## API

### Props

| Prop           | Тип       | По умолчанию       | Описание                                                        |
| -------------- | --------- | ------------------ | --------------------------------------------------------------- |
| `selector`     | `string`  | `'[data-reveal]'`  | CSS-селектор элементов внутри слота, которые нужно анимировать  |
| `visibleClass` | `string`  | `'is-visible'`     | Класс, который добавляется элементу при появлении               |
| `threshold`    | `number`  | `0.15`             | Доля видимости элемента, при которой срабатывает анимация (0..1) |
| `rootMargin`   | `string`  | `'0px 0px -10% 0px'` | Отступы вьюпорта (как в `IntersectionObserver`)               |
| `once`         | `boolean` | `true`             | Анимировать один раз или каждый раз при входе/выходе            |

### Пример с настройками

```vue
<ScrollReveal
  :threshold="0.3"
  :once="false"
  root-margin="0px 0px -20% 0px"
  visible-class="fade-in"
>
  <div data-reveal>...</div>
</ScrollReveal>
```

---

## Кастомизация анимации

Компонент **не навязывает** визуальный стиль — только добавляет класс `is-visible` при появлении.  
Анимация — в вашем CSS:

```css
[data-reveal] {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 0.6s ease, transform 0.6s ease;
}

[data-reveal].is-visible {
  opacity: 1;
  transform: translateY(0);
}
```

### Готовые рецепты

**Fade in:**
```css
[data-reveal] { opacity: 0; transition: opacity 0.5s; }
[data-reveal].is-visible { opacity: 1; }
```

**Slide from left:**
```css
[data-reveal] { opacity: 0; transform: translateX(-40px); transition: all 0.6s; }
[data-reveal].is-visible { opacity: 1; transform: translateX(0); }
```

**Scale up:**
```css
[data-reveal] { opacity: 0; transform: scale(0.9); transition: all 0.5s; }
[data-reveal].is-visible { opacity: 1; transform: scale(1); }
```

**Stagger-эффект (каскад):**
```css
[data-reveal]:nth-child(1) { transition-delay: 0ms; }
[data-reveal]:nth-child(2) { transition-delay: 100ms; }
[data-reveal]:nth-child(3) { transition-delay: 200ms; }
```

---

## Доступность

Компонент уважает `prefers-reduced-motion` — если пользователь отключил анимации в системе, всё будет показываться мгновенно:

```css
@media (prefers-reduced-motion: reduce) {
  [data-reveal] {
    opacity: 1 !important;
    transform: none !important;
    transition: none !important;
  }
}
```

Для браузеров без JS встроен fallback:

```css
@media (scripting: none) {
  [data-reveal] { opacity: 1 !important; transform: none !important; }
}
```

---

## Альтернативные способы использования

### Композабл `useReveal`

Если обёртка `<div>` ломает разметку (например, прямые дети `<ul>` или `<table>`):

```vue
<script setup>
import { ref } from 'vue'
import { useReveal } from '@/components/ScrollReveal.vue'

const listRef = ref(null)
useReveal({ root: () => listRef.value })
</script>

<template>
  <ul ref="listRef">
    <li data-reveal>Пункт 1</li>
    <li data-reveal>Пункт 2</li>
  </ul>
</template>
```

### Директива `v-reveal`

Самый лаконичный вариант:

```js
// main.js
import { createApp } from 'vue'
import { reveal } from '@/components/ScrollReveal.vue'
import App from './App.vue'

createApp(App).directive('reveal', reveal).mount('#app')
```

```vue
<template>
  <div v-reveal>Появится при скролле</div>
  <div v-reveal="{ threshold: 0.3, once: false }">С настройками</div>
</template>
```

---

## Совместимость

| Окружение              | Поддержка |
| ---------------------- | --------- |
| Vue 3                  | ✅         |
| Laravel + Inertia      | ✅         |
| SSR                    | ✅ (обсервер только в `onMounted`) |

---

## Как это работает под капотом

```js
onMounted(() => {
  const observer = new IntersectionObserver((entries) => {
    for (const entry of entries) {
      if (entry.isIntersecting) {
        entry.target.classList.add('is-visible')
        if (once) observer.unobserve(entry.target)
      }
    }
  }, { threshold, rootMargin })

  root.value
    ?.querySelectorAll(selector)
    .forEach(el => observer.observe(el))
})
```

- `onMounted` — гарантирует, что DOM уже отрисован (важно для SSR)
- `unobserve` после срабатывания — не держим обсерверы на уже анимированных элементах
- `onBeforeUnmount` — `disconnect()` на случай размонтирования компонента

---

## Лицензия

[MIT](./LICENSE) © [SuperLoca](https://github.com/SuperLoca)

---

<div align="center">

**Сделано с ❤️ для тех, кому лень писать `IntersectionObserver` в 100-й раз.**

[⭐ Поставить звезду](https://github.com/SuperLoca/ScrollRevealComponent) · [🐛 Сообщить о баге](https://github.com/SuperLoca/ScrollRevealComponent/issues) · [💡 Предложить идею](https://github.com/SuperLoca/ScrollRevealComponent/issues)

</div>