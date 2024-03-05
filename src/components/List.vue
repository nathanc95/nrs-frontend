<script>
export default {
  props: {
    isHighlighted: {
      type: Boolean,
      required: false
    },
    item: {
      required: true
    }
  }
};
</script>

<template>
  <li :class="{ 'highlighted': isHighlighted }">
    {{ item.state }}
  </li>
</template>

<style scoped>

</style>
