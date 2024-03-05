<script>
export default {
  props: {
    modelValue:{
      type: String,
      required: true
    }
  }
};
</script>

<template>
  <div>
    <input type="text"
           @input="e=>$emit('update:modelValue', e.target.value)"
           :value="modelValue"
           placeholder="filter by name">
  </div>
</template>

<style scoped>

</style>
