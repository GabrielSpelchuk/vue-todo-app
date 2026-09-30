<script setup>
import { ref, nextTick } from "vue";

const props = defineProps(["todo"]);
const emit = defineEmits(["delete", "update"]);

const editing = ref(false);
const titleField = ref(null);
const newTitle = ref(props.todo.title);

const startEditing = async () => {
  newTitle.value = props.todo.title;
  editing.value = true;

  await nextTick();

  if (titleField.value) {
    titleField.value.focus();
  }
};

const rename = () => {
  if (!editing.value) {
    return;
  }

  editing.value = true;

  if (newTitle.value === props.todo.title) {
    editing.value = false;
    return;
  }

  if (!newTitle.value) {
    editing.value = false;
    emit("delete");
    return;
  }

  emit("update", { ...props.todo, title: newTitle.value });

  editing.value = false;
};
</script>

<template>
  <div class="todo" :class="{ completed: todo.completed }">
    <label class="todo__status-label">
      <input
        type="checkbox"
        class="todo__status"
        :checked="todo.completed"
        @change="emit('update', { ...todo, completed: !todo.completed })"
      />
    </label>

    <form v-if="editing" @submit.prevent="rename">
      <input
        @keyup.escape="editing = false"
        class="todo__title-field"
        placeholder="Empty todo will be deleted"
        v-model.trim="newTitle"
        ref="titleField"
        @blur="rename"
      />
    </form>

    <template v-else>
      <span class="todo__title" @dblclick="startEditing">{{ todo.title }}</span>
      <button class="todo__remove" @click="emit('delete')">x</button>
    </template>

    <div class="modal overlay" :class="{ 'is-active': false }">
      <div class="modal-background has-background-white-ter">
        <div class="loader"></div>
      </div>
    </div>
  </div>
</template>
