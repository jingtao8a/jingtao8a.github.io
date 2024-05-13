---
title: vue3学习——Tutorial
date: 2024-05-12 21:00:54
tags: vue3
categories: vue3 
---


### reactive() 和 ref()
```javascript
<script setup>
import { reactive, ref } from 'vue'

const counter = reactive({ count: 0 })
const message = ref('Hello World!')
</script>

<template>
  <h1>{{ message }}</h1>
  <h1>{{ message.split('').reverse().join('') }}</h1>
  <p>Count is: {{ counter.count }}</p>
</template>
```

### v-bind
v-bind:id=""<br>
v-bind:class=""<br>
v-bind:value=""<br>

简化为<br>

:id=""<br>
:class=""<br>
:value=""<br>

```javascript
<script setup>
import { ref } from 'vue'

const titleClass = ref('title')
</script>

<template>
  <h1 :class="titleClass">Make me red</h1>
</template>

<style>
.title {
  color: red;
}
</style>
```

### v-on
Event Listeners

v-on:click=""<br>
v-on:input=""<br>

简化为<br>

@click=""<br>
@input=""<br>

```javascript
<script setup>
import { ref } from 'vue'

const count = ref(0)

function increment() {
  count.value++
}
</script>

<template>
  <button @click="increment">Count is: {{ count }}</button>
</template>
```

### v-model

```javascript
<script setup>
import { ref } from 'vue'

const text = ref('')
</script>

<template>
  <input v-model="text" placeholder="Type here">
  <p>{{ text }}</p>
</template>
```

### v-if 和 v-else

Conditional Rendering

```javascript
<script setup>
import { ref } from 'vue'

const awesome = ref(true)

function toggle() {
  awesome.value = !awesome.value
}
</script>

<template>
  <button @click="toggle">Toggle</button>
  <h1 v-if="awesome">Vue is awesome!</h1>
  <h1 v-else>Oh no 😢</h1>
</template>
```

### v-for

List Rendering

```javascript
<script setup>
import { ref } from 'vue'

// give each todo a unique id
let id = 0

const newTodo = ref('')
const todos = ref([
  { id: id++, text: 'Learn HTML' },
  { id: id++, text: 'Learn JavaScript' },
  { id: id++, text: 'Learn Vue' }
])
// const index = ref("123")
function addTodo() {
  // ...
  todos.value.push({id: id++, text: newTodo.value});
  newTodo.value = '';
}

function removeTodo(todo) {
  // ...
  // index.value
  todos.value.splice(todos.value.indexOf(todo), 1)
}
</script>

<template>
  <form @submit.prevent="addTodo">
    <input v-model="newTodo" required placeholder="new todo">
    <button>Add Todo</button>
  </form>
<!--   <p>
    {{index}}
  </p> -->
  <ul>
    <li v-for="todo in todos" :key="todo.id">
      {{ todo.text }}
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
</template>
```


### computed

> introducing computed().We can create a computed ref that computes its .value based on other reactive data sources:

```javascript
<script setup>
import { ref, computed } from 'vue'

let id = 0

const newTodo = ref('')
const hideCompleted = ref(false)
const todos = ref([
  { id: id++, text: 'Learn HTML', done: true },
  { id: id++, text: 'Learn JavaScript', done: true },
  { id: id++, text: 'Learn Vue', done: false }
])

const filteredTodos = computed(() => {
  return hideCompleted.value
    ? todos.value.filter((t) => !t.done)
    : todos.value
})

function addTodo() {
  todos.value.push({ id: id++, text: newTodo.value, done: false })
  newTodo.value = ''
}

function removeTodo(todo) {
  todos.value = todos.value.filter((t) => t !== todo)
}
</script>

<template>
  <form @submit.prevent="addTodo">
    <input v-model="newTodo" required placeholder="new todo">
    <button>Add Todo</button>
  </form>
  <ul>
    <li v-for="todo in filteredTodos" :key="todo.id">
      <input type="checkbox" v-model="todo.done">
      <span :class="{ done: todo.done }">{{ todo.text }}</span>
      <button @click="removeTodo(todo)">X</button>
    </li>
  </ul>
  <button @click="hideCompleted = !hideCompleted">
    {{ hideCompleted ? 'Show all' : 'Hide completed' }}
  </button>
</template>

<style>
.done {
  text-decoration: line-through;
}
</style>
```

### Lifecycle and Template Refs

> Hook函数 onMounted onUpdated 等 ...

```javascript
<script setup>
import { ref, onMounted } from 'vue'

const pElementRef = ref(null)
const customValue = ref("123")
onMounted(() => {
  customValue.value = pElementRef.value.textContent
  pElementRef.value.textContent = 'mounted!'
})
</script>

<template>
  <p> {{customValue}} </p>
  <p ref="pElementRef">Hello</p>
</template>
```

### watch
```javascript
import { ref, watch } from 'vue'

const count = ref(0)

watch(count, (newCount) => {
  // yes, console.log() is a side effect
  console.log(`new count is: ${newCount}`)
})
```
> watch() can directly watch a ref, and the callback gets fired whenever count's value changes.

```javascript
<script setup>
import { ref, watch } from 'vue'

const todoId = ref(1)
const todoData = ref(null)

async function fetchData(id) {
  todoData.value = null
  const res = await fetch(
    `https://jsonplaceholder.typicode.com/todos/${id.value}`
  )
  todoData.value = await res.json()
}
  
fetchData(todoId)
watch(todoId, (todoId) => {
  fetchData(todoId);
})

  
</script>

<template>
  <p>Todo id: {{ todoId }}</p>
  <button @click="todoId++" :disabled="!todoData">Fetch next todo</button>
  <p v-if="!todoData">Loading...</p>
  <pre v-else>{{ todoData }}</pre>
</template>
```

### Components
```javascript
<script setup>
import ChildComp from './ChildComp.vue'
</script>

<template>
  <!-- render child component -->
  <ChildComp></ChildComp>
</template>
```

### Props
> A child component can accept input from the parent via props. First, it needs to declare the props it accepts:

App.vue
```javascript
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const greeting = ref('Hello from parent')
</script>

<template>
  <ChildComp :msg="greeting" />
</template>
```

ChildComp.vue
```javascript
<script setup>
const props = defineProps({
  msg: String
})
</script>

<template>
  <h2>{{ msg || 'No props passed yet' }}</h2>
</template>
```

### Emits
> In addition to receiving props, a child component can also emit events to the parent:

```App.vue
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const childMsg = ref('No child msg yet')
</script>

<template>
  <ChildComp @response="(msg) => childMsg = msg " @response1="(msg)=> childMsg=msg"/>
  <p>{{ childMsg }}</p>
</template>
```

```ChildComp.Vue
<script setup>
const emit = defineEmits(['response', 'response1'])

emit('response', 'hello from child')
emit('response1', "flafkjka")
</script>

<template>
  <h2>Child component</h2>
</template>
```

### Slots
> In addition to passing data via props, the parent component can also pass down template fragments to the child via slots

App.vue
```javascript
<script setup>
import { ref } from 'vue'
import ChildComp from './ChildComp.vue'

const msg = ref('from parent')
</script>

<template>
  <ChildComp></ChildComp>
</template>
```

ChildComp.vue
```javascript
<template>
  <slot>Fallback content</slot>
</template>
```

