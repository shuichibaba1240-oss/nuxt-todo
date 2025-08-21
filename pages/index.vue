<!-- /pages/index.vue -->
<template>
  <main class="container">
    <header class="head">
      <h1>タスク管理（ミニ）</h1>
      <p class="sub">追加 → 検索 → 完了チェック → 削除。ブラウザに自動保存。</p>
    </header>

    <section class="controls">
      <form class="add" @submit.prevent="add">
        <input v-model="newTitle" placeholder="やることを入力…" />
        <button :disabled="!newTitle.trim()">追加</button>
      </form>

      <div class="filters">
        <input v-model="q" placeholder="検索（例：買い物）" />
        <div class="seg">
          <label
            ><input type="radio" value="all" v-model="filter" />すべて</label
          >
          <label
            ><input type="radio" value="active" v-model="filter" />未完</label
          >
          <label
            ><input type="radio" value="done" v-model="filter" />完了</label
          >
        </div>
      </div>
    </section>

    <ul class="list" v-if="visible.length">
      <li v-for="t in visible" :key="t.id">
        <label class="row">
          <input type="checkbox" v-model="t.done" />
          <span :class="{ done: t.done }">{{ t.title }}</span>
        </label>
        <button class="del" @click="remove(t.id)">削除</button>
      </li>
    </ul>
    <p v-else class="empty">まだ何もありません</p>

    <footer class="foot">
      <span>合計: {{ todos.length }}</span>
      <span>完了: {{ doneCount }}</span>
      <button class="ghost" @click="clearCompleted" :disabled="doneCount === 0">
        完了一括削除
      </button>
    </footer>
  </main>
</template>

<script setup lang="ts">
import { ref, computed, watch, onMounted } from 'vue';

type Todo = { id: string; title: string; done: boolean };
type Filter = 'all' | 'active' | 'done';

const STORAGE_KEY = 'todos-v1';
const todos = ref<Todo[]>([]);
const newTitle = ref('');
const q = ref('');
const filter = ref<Filter>('all');

onMounted(() => {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    todos.value = raw ? (JSON.parse(raw) as Todo[]) : [];
  } catch {
    todos.value = [];
  }
});

watch(
  todos,
  (val) => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(val));
  },
  { deep: true }
);

function add() {
  const title = newTitle.value.trim();
  if (!title) return;
  todos.value.unshift({ id: crypto.randomUUID(), title, done: false });
  newTitle.value = '';
}
function remove(id: string) {
  todos.value = todos.value.filter((t) => t.id !== id);
}
function clearCompleted() {
  todos.value = todos.value.filter((t) => !t.done);
}

const doneCount = computed(() => todos.value.filter((t) => t.done).length);

const visible = computed(() => {
  const query = q.value.trim().toLowerCase();
  return todos.value.filter((t) => {
    const matchQuery = !query || t.title.toLowerCase().includes(query);
    const matchFilter =
      filter.value === 'all' ||
      (filter.value === 'active' && !t.done) ||
      (filter.value === 'done' && t.done);
    return matchQuery && matchFilter;
  });
});
</script>

<style scoped>
:root {
  color-scheme: light dark;
}
.container {
  max-width: 720px;
  margin: 2rem auto;
  padding: 1rem;
}
.head h1 {
  margin: 0 0 0.25rem;
  font-size: 1.6rem;
}
.sub {
  margin: 0 0 1rem;
  opacity: 0.7;
  font-size: 0.9rem;
}
.controls {
  display: grid;
  gap: 0.75rem;
  margin-bottom: 0.75rem;
}
.add {
  display: flex;
  gap: 0.5rem;
}
.add input {
  flex: 1;
  padding: 0.6rem 0.7rem;
}
.add button {
  padding: 0.6rem 0.9rem;
  cursor: pointer;
}
.filters {
  display: grid;
  gap: 0.5rem;
}
.filters input {
  padding: 0.5rem 0.6rem;
}
.seg {
  display: inline-flex;
  gap: 1rem;
  align-items: center;
}
.list {
  list-style: none;
  padding: 0;
  margin: 0.5rem 0 0;
  display: grid;
  gap: 0.5rem;
}
.row {
  display: flex;
  align-items: center;
  gap: 0.6rem;
  flex: 1;
}
li {
  display: flex;
  align-items: center;
  gap: 0.5rem;
  padding: 0.6rem 0.7rem;
  border: 1px solid rgba(127, 127, 127, 0.25);
  border-radius: 0.5rem;
}
.del {
  padding: 0.3rem 0.6rem;
  cursor: pointer;
}
.done {
  text-decoration: line-through;
  opacity: 0.6;
}
.empty {
  opacity: 0.7;
  text-align: center;
  padding: 1.5rem 0;
}
.foot {
  display: flex;
  gap: 1rem;
  justify-content: space-between;
  align-items: center;
  margin-top: 1rem;
  font-size: 0.95rem;
}
.ghost {
  background: transparent;
  border: 1px solid rgba(127, 127, 127, 0.4);
  padding: 0.4rem 0.7rem;
  border-radius: 0.4rem;
  cursor: pointer;
}
@media (max-width: 480px) {
  .container {
    padding: 0.75rem;
  }
  .seg {
    gap: 0.75rem;
    font-size: 0.95rem;
  }
}
</style>
