# Vue.js

Vue.js is a progressive JavaScript framework for building user interfaces.

## Quick Start

```bash
# Create new Vue app
npm create vue@latest my-app
cd my-app
npm install
npm run dev
```

## Component Basics

### Single File Component (SFC)

```vue
<template>
  <div class="greeting">
    <h1>{{ message }}</h1>
    <button @click="reverseMessage">Reverse</button>
  </div>
</template>

<script setup>
import { ref } from 'vue';

const message = ref('Hello Vue!');

const reverseMessage = () => {
  message.value = message.value.split('').reverse().join('');
};
</script>

<style scoped>
.greeting {
  padding: 20px;
}
</style>
```

## Reactivity

### ref

```javascript
import { ref } from 'vue';

const count = ref(0);
console.log(count.value); // 0

count.value++;
console.log(count.value); // 1
```

### reactive

```javascript
import { reactive } from 'vue';

const state = reactive({
  count: 0,
  message: 'Hello'
});

state.count++;
```

### computed

```javascript
import { ref, computed } from 'vue';

const firstName = ref('John');
const lastName = ref('Doe');

const fullName = computed(() => {
  return `${firstName.value} ${lastName.value}`;
});
```

### watch

```javascript
import { ref, watch } from 'vue';

const count = ref(0);

watch(count, (newValue, oldValue) => {
  console.log(`Count changed from ${oldValue} to ${newValue}`);
});
```

## Template Syntax

### Text Interpolation

```vue
<template>
  <p>{{ message }}</p>
  <p v-text="message"></p>
</template>
```

### Attribute Binding

```vue
<template>
  <div :id="dynamicId" :class="{ active: isActive }">
    <img :src="imageUrl" :alt="imageAlt" />
  </div>
</template>
```

### Event Handling

```vue
<template>
  <button @click="handleClick">Click me</button>
  <button @click="increment(5)">Add 5</button>
  <input @keyup.enter="submit" />
</template>

<script setup>
const handleClick = () => {
  console.log('Clicked!');
};

const increment = (n) => {
  count.value += n;
};
</script>
```

## Conditional Rendering

```vue
<template>
  <div v-if="isVisible">Visible</div>
  <div v-else-if="isHidden">Hidden</div>
  <div v-else>Default</div>

  <div v-show="showElement">Toggle with CSS</div>
</template>
```

## List Rendering

```vue
<template>
  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.name }}
    </li>
  </ul>

  <div v-for="(value, key, index) in object" :key="key">
    {{ index }}. {{ key }}: {{ value }}
  </div>
</template>
```

## Form Input Bindings

```vue
<template>
  <input v-model="message" placeholder="Enter message" />
  <p>Message: {{ message }}</p>

  <input type="checkbox" v-model="checked" />

  <select v-model="selected">
    <option value="A">Option A</option>
    <option value="B">Option B</option>
  </select>
</template>

<script setup>
import { ref } from 'vue';

const message = ref('');
const checked = ref(false);
const selected = ref('');
</script>
```

## Component Communication

### Props

```vue
<!-- Parent -->
<template>
  <ChildComponent :message="parentMessage" :count="10" />
</template>

<!-- Child -->
<script setup>
const props = defineProps({
  message: String,
  count: {
    type: Number,
    required: true,
    default: 0
  }
});
</script>
```

### Emits

```vue
<!-- Child -->
<template>
  <button @click="handleClick">Click</button>
</template>

<script setup>
const emit = defineEmits(['update', 'delete']);

const handleClick = () => {
  emit('update', { id: 1, value: 'new' });
};
</script>

<!-- Parent -->
<template>
  <ChildComponent @update="handleUpdate" />
</template>
```

## Lifecycle Hooks

```vue
<script setup>
import { onMounted, onUpdated, onUnmounted } from 'vue';

onMounted(() => {
  console.log('Component mounted');
});

onUpdated(() => {
  console.log('Component updated');
});

onUnmounted(() => {
  console.log('Component unmounted');
});
</script>
```

## Composables

```javascript
// useCounter.js
import { ref } from 'vue';

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);

  const increment = () => count.value++;
  const decrement = () => count.value--;
  const reset = () => count.value = initialValue;

  return {
    count,
    increment,
    decrement,
    reset
  };
}

// Using in component
import { useCounter } from './useCounter';

const { count, increment, decrement } = useCounter(10);
```

## Vue Router

```javascript
import { createRouter, createWebHistory } from 'vue-router';

const router = createRouter({
  history: createWebHistory(),
  routes: [
    { path: '/', component: Home },
    { path: '/about', component: About },
    { path: '/user/:id', component: User }
  ]
});
```

```vue
<template>
  <router-link to="/">Home</router-link>
  <router-link to="/about">About</router-link>

  <router-view />
</template>
```

## Pinia (State Management)

```javascript
// stores/counter.js
import { defineStore } from 'pinia';

export const useCounterStore = defineStore('counter', {
  state: () => ({
    count: 0
  }),
  getters: {
    doubleCount: (state) => state.count * 2
  },
  actions: {
    increment() {
      this.count++;
    }
  }
});

// Using in component
import { useCounterStore } from '@/stores/counter';

const counter = useCounterStore();
counter.increment();
```

## Resources

- [Official Documentation](https://vuejs.org/)
- [Vue School](https://vueschool.io/)
- [Vue Mastery](https://www.vuemastery.com/)
