<template>
  <div class="form-group" :class="{ 'form-group_inline': inline }">
    <!-- form-group_inline -->
    <label v-if="label" class="form-group__label">{{ label }}</label>
    <slot />
    <!-- CONTENT -->
  </div>
</template>

<script lang="ts">
import { defineComponent } from 'vue';

export default defineComponent({
  name: 'UiFormGroup',
  props: {
    label: String,
    inline: {
      type: Boolean,
      required: true,
    },
  },
});
</script>

<style scoped>
.form-group {
  position: relative;
  margin-bottom: 24px;
}

.form-group.form-group_inline {
  display: inline-block;
  margin-bottom: 0;
}

.form-group.form-group_inline + .form-group.form-group_inline {
  margin-left: 16px;
}

.form-group__label {
  font-weight: 400;
  font-size: 20px;
  line-height: 28px;
  color: var(--body-color);
  margin-bottom: 10px;
  display: block;
}
</style>
