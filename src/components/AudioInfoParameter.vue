<template>
  <div class="q-mx-md">
    <div>
      <div
        class="text-body1 q-mb-xs"
        :class="{
          disabled: slider.qSliderProps.disable.value,
        }"
      >
        {{ label }}
        <span class="editable">
          {{ slider.state.currentValue.value?.toFixed(2) }}
          <popup-number-edit
            :value="
              slider.state.currentValue.value
                ? slider.state.currentValue.value.toFixed(2)
                : '0'
            "
            @update="updateFromTextInput"
          />
        </span>
      </div>
    </div>
    <q-slider
      dense
      snap
      color="primary-light"
      trackSize="2px"
      :min="slider.qSliderProps.min.value"
      :max="slider.qSliderProps.max.value"
      :step="slider.qSliderProps.step.value"
      :disable="slider.qSliderProps.disable.value"
      :model-value="slider.qSliderProps.modelValue.value"
      @update:model-value="slider.qSliderProps['onUpdate:modelValue']"
      @change="slider.qSliderProps.onChange"
      @wheel="slider.qSliderProps.onWheel"
      @pan="slider.qSliderProps.onPan"
    />
  </div>
</template>

<script lang="ts">
import { PreviewSliderHelper } from "@/helpers/previewSliderHelper";
import PopupNumberEdit from "./PopupNumberEdit.vue";
import { defineComponent, PropType } from "vue";

export default defineComponent({
  name: "AudioInfoParameter",
  components: { PopupNumberEdit },

  props: {
    label: { type: String, required: true },
    slider: { type: Object as PropType<PreviewSliderHelper>, required: true },
    onChange: {
      type: Function as PropType<(newValue: number) => void>,
      required: true,
    },
  },

  setup(props) {
    const updateFromTextInput = (newValue: string) => {
      const parsedInput = parseFloat(newValue);
      if (Number.isNaN(parsedInput)) {
        return;
      }
      const clamped = Math.min(
        Math.max(parsedInput, props.slider.qSliderProps.min.value),
        props.slider.qSliderProps.max.value
      );
      props.onChange(clamped);
    };

    return {
      updateFromTextInput,
    };
  },
});
</script>

<style scoped lang="scss">
.editable {
  cursor: text;
}
</style>
