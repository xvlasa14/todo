<script setup>
import { ref, computed } from "vue";
import tasksData from "../public/data/tasks.json";

const data = ref(tasksData);
const plannedOnly = ref(false);

const tasks = computed(() => {
  let result = [...data.value];

  if (plannedOnly.value) {
    result = result.filter((task) => task.status === "planned");
  }

  return result.sort((a, b) => {
    if (a.status !== b.status) {
      return a.status === "planned" ? -1 : 1;
    }

    const priorityOrder = {
      high: 3,
      normal: 2,
      low: 1,
    };

    return priorityOrder[b.priority] - priorityOrder[a.priority];
  });
});

const completeTask = (id) => {
  const task = data.value.find((t) => t.id === id);
  if (task) {
    task.status = "finished";
  }
};

const deleteTask = (id) => {
  data.value = data.value.filter((t) => t.id !== id);
};

const summary = computed(() => {
  const tasksTotal = data.value.length;
  const tasksFinished = data.value.filter(
    (t) => t.status === "finished",
  ).length;

  return {
    tasksTotal,
    tasksFinished,
  };
});
</script>
<template>
  <div>
    <div>
      <h1>Moje úkoly</h1>
      <button>Nový úkol</button>
      <label>
        <input type="checkbox" v-model="plannedOnly" />
        Pouze nedokončené
      </label>
    </div>

    <div v-for="task in tasks" :key="task.id">
      <h3>{{ task.title }}</h3>
      <p>Priorita: {{ task.priority }}</p>
      <p>Stav: {{ task.status }}</p>

      <button v-if="task.status === 'planned'" @click="completeTask(task.id)">
        Dokončit
      </button>

      <button @click="deleteTask(task.id)">Odstranit</button>
    </div>

    <p>Celkem úkolů: {{ summary.tasksTotal }}</p>
    <p>Dokončeno: {{ summary.tasksFinished }}</p>
  </div>
</template>
