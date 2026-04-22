<script setup>
import { ref, computed, watch } from "vue";
import tasksData from "../public/data/tasks.json";
import templatesData from "../public/data/templates.json";

const data = ref(tasksData);

const plannedOnly = ref(false);

const isModalOpen = ref(false);
const newTaskTemplate = ref(false);
const newTaskTitle = ref("");
const newTaskPriority = ref("normal");

const templateId = ref("");

const priorityOrder = {
  high: 3,
  normal: 2,
  low: 1,
};

const tasks = computed(() => {
  let result = [...data.value];

  if (plannedOnly.value) {
    result = result.filter((task) => task.status === "planned");
  }

  return result.sort((a, b) => {
    if (a.status !== b.status) {
      return a.status === "planned" ? -1 : 1;
    }
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

const addTask = () => {
  if (!newTaskTitle.value.trim()) return;

  data.value.push({
    id: Date.now(),
    title: newTaskTitle.value,
    status: "planned",
    priority: newTaskPriority.value,
  });

  newTaskTitle.value = "";
  newTaskPriority.value = "normal";

  isModalOpen.value = false;
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

watch(templateId, (newId) => {
  const template = templatesData.find((t) => t.id === newId);
  if (!template) return;

  newTaskTitle.value = template.defaults.title;
  newTaskPriority.value = template.defaults.priority;
});
</script>

<template>
  <div class="w-3/5 mx-auto h-dvh overflow-hidden">
    <h1 class="text-3xl text-center mx-auto my-3 block">Moje úkoly</h1>

    <div class="flex flex-row justify-between mb-5">
      <button
        class="bg-emerald-200 py-1 px-2 rounded-sm text-emerald-900 font-semibold"
        @click="isModalOpen = true"
      >
        Nový úkol
      </button>
      <label class="flex flex-row gap-2">
        <input type="checkbox" v-model="plannedOnly" />
        Pouze nedokončené
      </label>
    </div>
    <div class="flex flex-col gap-5 overflow-scroll h-3/4">
      <div
        v-for="task in tasks"
        :key="task.id"
        class="p-3 border-l-2"
        :class="{
          'border-emerald-700 bg-emerald-50 text-emerald-900':
            task.priority === 'low',
          'border-amber-700 bg-amber-50 text-amber-900':
            task.priority === 'normal',
          'border-red-700 bg-red-50 text-red-900': task.priority === 'high',
          'opacity-50': task.status === 'finished',
        }"
      >
        <h3 class="font-semibold">{{ task.title }}</h3>
        <p>Priorita: {{ task.priority }}</p>
        <p>Stav: {{ task.status }}</p>

        <div class="flex flex-row gap-3 my-3 justify-end">
                <button class="bg-stone-50 hover:bg-stone-100 px-2 py-1 text-zinc-900 rounded-sm border border-stone-300 hover:border-stone-500 mr-auto" v-if="task.status === 'planned'" @click="completeTask(task.id)">Dokončit</button>

        <button class="underline px-2 py-1 text-zinc-900 hover:text-zinc-500" @click="deleteTask(task.id)">Odstranit</button>
  
        </div>
      </div>
    </div>

    <p>Celkem úkolů: {{ summary.tasksTotal }}</p>
    <p>Dokončeno: {{ summary.tasksFinished }}</p>
  </div>

  <Teleport to="#teleports">
    <div v-if="isModalOpen" class="absolute size-full top-1 flex m-auto bg-slate-500/60 overflow-hidden">
      <div class="w-2/5 h-2/5 p-10 bg-slate-200 m-auto">
        <h2 class="text-2xl text-center">Nový úkol</h2>
        <div class="flex flex-col gap-1 my-2">
<h3 class="font-semibold">Šablona</h3>
        <select v-model="templateId" class="bg-slate-50 px-1 py-1">
          <option disabled value="">-- žádná šablona --</option>
          <option
            v-for="template in templatesData"
            :key="template.id"
            :value="template.id"
          >
            {{ template.title }}
          </option>
        </select>
        </div>
        
        <div class="flex flex-col gap-1 my-3">
          <h3 class="font-semibold">Název úkolu</h3>
          <input v-model="newTaskTitle" placeholder="Název úkolu" />
        </div>
        <div class="flex flex-col gap-1 my-3">
          <h3 class="font-semibold">Priorita</h3>
        <div class="flex flex-row gap-3 items-center">
          <label v-for="(key, val) in priorityOrder">
            <input type="radio" :value="val" v-model="newTaskPriority" />
            {{ val }}
          </label>
        </div>
      </div>
      <div class="flex flex-row gap-3 my-3 justify-end">
        <button class="bg-stone-50 hover:bg-stone-100 px-2 py-1 text-zinc-900 rounded-sm border border-stone-300 hover:border-stone-500 mr-auto" @click="addTask">Přidat</button>
        <button class="underline px-2 py-1 text-zinc-900 hover:text-zinc-500" @click="isModalOpen = false">Zrušit</button>
      </div>
      </div>
    </div>
  </Teleport>
</template>
