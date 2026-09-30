<script setup>
defineProps({ hidden: Boolean });
defineEmits(['close']);
</script>

<template>
  <article class="message" :class="{ 'message--hidden': hidden }">
    <div class="message-header">
      <slot name="header">Message</slot>
      <button class="delete" @click="$emit('close')"></button>
    </div>

    <div class="message-body">
      <slot> No content </slot>
    </div>
  </article>
</template>

<style scoped lang="scss">
.message {
  transform-origin: top center;
  transition-property: opacity, transform;
  transition-duration: 0.3s;

  &--hidden {
    transform: scaleY(0);
    opacity: 0;
    pointer-events: none;
  }
}
</style>
