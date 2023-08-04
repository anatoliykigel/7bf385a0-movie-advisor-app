<script setup>
defineProps({
  groupTitle: String,
  itemsList: Object,
  itemModel: [String, Number],
  endItemsWith: {
    type: String,
    default: ''
  }
})

defineEmits(['update:itemModel'])
</script>

<template>
  <div class="main-form__control-section__control-group">
    <h4>{{ groupTitle }}</h4>

    <select :value="itemModel" @input="$emit('update:itemModel', $event.target.value)">
      <option v-for="(item, i) in itemsList" :key="item" :value="item">
        {{ item }}
        <span v-if="i != 0">
          {{ endItemsWith }}
        </span>
      </option>
    </select>
  </div>
</template>

<style lang="scss" scoped></style>
