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
    const template = templatesData.find(t => t.id === newId)
    if (!template) return

    newTaskTitle.value = template.defaults.title
    newTaskPriority.value = template.defaults.priority
  })
</script>

<template>
  <div>
    <div>
      <h1 class="">Moje úkoly</h1>
      <button class="" @click="isModalOpen = true">Nový úkol</button>
      <label>
        <input type="checkbox" v-model="plannedOnly" />
        Pouze nedokončené
      </label>
    </div>

    <div v-for="task in tasks" :key="task.id">
      <h3>{{ task.title }}</h3>
      <p>Priorita: {{ task.priority }}</p>
      <p>Stav: {{ task.status }}</p>

      <button class="" v-if="task.status === 'planned'" @click="completeTask(task.id)">
        Dokončit
      </button>

      <button class="" @click="deleteTask(task.id)">Odstranit</button>
    </div>

    <p>Celkem úkolů: {{ summary.tasksTotal }}</p>
    <p>Dokončeno: {{ summary.tasksFinished }}</p>
  </div>

  <Teleport to="#teleports">
    <div v-if="isModalOpen" class="modal d-block bg-secondary bg-opacity-25">
      <div>
        <h2>Nový úkol</h2>
        <h3>Vyber šablonu</h3>

        <select v-model="templateId">
          <option disabled value="">-- žádná šablona --</option>

          <option
            v-for="template in templatesData"
            :key="template.id"
            :value="template.id"
          >
            {{ template.title }}
          </option>
        </select>
        <input v-model="newTaskTitle" placeholder="Název úkolu" />

        <div>
          <label v-for="(key, val) in priorityOrder">
            <input type="radio" :value="val" v-model="newTaskPriority" />
            {{ val }}
          </label>
        </div>

        <button class="" @click="addTask">Přidat</button>
        <button class="" @click="isModalOpen = false">Zrušit</button>
      </div>
    </div>
  </Teleport>
</template>
