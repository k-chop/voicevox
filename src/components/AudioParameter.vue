<template>
  <div
    @mouseenter="handleMouseHover(true)"
    @mouseleave="handleMouseHover(false)"
  >
    <q-badge
      class="value-label"
      color="primary-light"
      text-color="display-on-primary"
      v-if="
        !directInputMode &&
        !disable &&
        (valueLabel.visible || previewSlider.state.isPanning.value)
      "
    >
      {{ previewSlider.state.currentValue.value?.toFixed(precisionComputed) }}
    </q-badge>
    <q-badge
      class="value-label clickable"
      color="primary-light"
      text-color="display-on-primary"
      v-if="directInputMode"
    >
      {{ previewSlider.state.currentValue.value?.toFixed(precisionComputed) }}
      <q-popup-edit
        class="number-edit"
        :model-value="
          previewSlider.state.currentValue.value?.toFixed(precisionComputed)
        "
        @update:model-value="changeValueFromTextInput"
        auto-save
        transition-show="none"
        transition-hide="none"
        v-slot="scope"
        max-width="48px"
        cover
      >
        <q-input
          center
          v-model.number="scope.value"
          dense
          autofocus
          outlined
          input-class="text-center"
          input-style="padding: 0"
          @keyup.enter="scope.set"
          @focus="(input) => input.target.select()"
        />
      </q-popup-edit>
    </q-badge>
    <q-slider
      vertical
      reverse
      snap
      color="primary-light"
      trackSize="2.5px"
      :style="clipPathComputed"
      :min="previewSlider.qSliderProps.min.value"
      :max="previewSlider.qSliderProps.max.value"
      :step="previewSlider.qSliderProps.step.value"
      :disable="previewSlider.qSliderProps.disable.value"
      :model-value="previewSlider.qSliderProps.modelValue.value"
      @update:model-value="previewSlider.qSliderProps['onUpdate:modelValue']"
      @click.stop="
        undefined; // クリックでアクセント句が選択されないように
      "
      @change="previewSlider.qSliderProps.onChange"
      @wheel="previewSlider.qSliderProps.onWheel"
      @pan="previewSlider.qSliderProps.onPan"
    />
  </div>
</template>

<script lang="ts">
import { previewSliderHelper } from "@/helpers/previewSliderHelper";
import { MoraDataType } from "@/type/preload";
import { computed, defineComponent, reactive } from "vue";

export default defineComponent({
  name: "AudioParameter",

  props: {
    value: { type: Number, required: true },
    accentPhraseIndex: { type: Number, required: true },
    moraIndex: { type: Number, required: true },
    uiLocked: { type: Boolean, required: true },
    min: { type: Number, default: 0.0 },
    max: { type: Number, default: 10.0 },
    step: { type: Number, default: 0.01 },
    disable: { type: Boolean, default: false },
    type: { type: String as () => MoraDataType, default: "vowel" },
    clip: { type: Boolean, default: false },
    shiftKeyFlag: { type: Boolean, default: false },
    directInputMode: { type: Boolean, default: false },
  },

  emits: ["changeValue", "mouseOver"],

  setup(props, { emit }) {
    const changeValue = (newValue: number, type: MoraDataType = props.type) => {
      emit(
        "changeValue",
        props.accentPhraseIndex,
        props.moraIndex,
        newValue,
        type
      );
    };

    const changeValueFromTextInput = (newValue: string) => {
      const newNumber = parseFloat(newValue);
      if (Number.isNaN(newNumber)) {
        return;
      }
      const clamped = Math.min(Math.max(newNumber, props.min), props.max);
      changeValue(clamped);
    };

    const previewSlider = previewSliderHelper({
      modelValue: () => props.value,
      disable: () => props.disable || props.uiLocked,
      onChange: changeValue,
      max: () => props.max,
      min: () => props.min,
      step: () => props.step,
      scrollStep: () => props.step * 10,
      scrollMinStep: () => props.step,
      disableScroll: () => props.shiftKeyFlag, // shift+ホイール操作の横方向スクロール中にスライダー操作を無視するため
    });

    const valueLabel = reactive({
      visible: false,
    });

    const clipPathComputed = computed((): string => {
      if (!props.clip) {
        return "";
      } else {
        if (props.type == "vowel") {
          return "clip-path: inset(-50% -50% -50% 50%)";
        } else {
          return "clip-path: inset(-50% 50% -50% -50%)";
        }
      }
    });

    const handleMouseHover = (isOver: boolean) => {
      valueLabel.visible = isOver;
      if (props.type == "consonant" || props.type == "vowel") {
        emit(
          "mouseOver",
          isOver,
          props.type,
          props.accentPhraseIndex,
          props.moraIndex
        );
      }
    };

    const precisionComputed = computed(() => {
      if (props.type == "pause" || props.type == "pitch") {
        return 2;
      } else {
        return 3;
      }
    });

    return {
      previewSlider,
      changeValue,
      changeValueFromTextInput,
      valueLabel,
      clipPathComputed,
      handleMouseHover,
      precisionComputed,
    };
  },
});
</script>

<style scoped lang="scss">
$value-label-height: 24px;

div {
  position: absolute;
  top: 8px;
  bottom: 8px;
  .q-slider {
    height: calc(100% - #{$value-label-height + 12px});
    margin-top: $value-label-height + 12px;
    min-width: 20px;
    max-width: 20px;
    :deep(.q-slider__track-container--v) {
      margin-left: -3.5px;
    }
  }
  .value-label {
    height: $value-label-height;
    padding: 0px 8px;
    transform: translateX(-50%) translateX(15px);
    z-index: 3;

    &.clickable {
      cursor: text;
    }
  }
}

:deep(.number-edit) {
  padding: 0;

  .q-field__control {
    padding: 0;
  }
}
</style>
