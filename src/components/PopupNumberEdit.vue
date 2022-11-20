<template>
  <q-popup-edit
    class="popup-number-edit"
    :model-value="value"
    @update:model-value="onUpdate"
    auto-save
    transition-show="none"
    transition-hide="none"
    v-slot="scope"
    max-width="48px"
    cover
  >
    <q-input
      center
      v-model="scope.value"
      dense
      autofocus
      outlined
      input-class="text-center"
      input-style="padding: 0"
      @keyup.enter="scope.set"
      @focus="onFocus"
    />
  </q-popup-edit>
</template>

<script lang="ts">
import { defineComponent } from "vue";

export default defineComponent({
  name: "PopupNumberEdit",

  emits: ["update"],

  props: {
    value: { type: String, required: true },
  },

  setup(_, { emit }) {
    const onFocus = (input: FocusEvent) => {
      input && input.target && (input.target as HTMLInputElement).select();
    };

    const onUpdate = (newValue: string) => {
      emit("update", newValue);
    };

    return {
      onFocus,
      onUpdate,
    };
  },
});
</script>

<style lang="scss">
.popup-number-edit {
  padding: 0;

  .q-field__control {
    padding: 0;
  }
}
</style>
