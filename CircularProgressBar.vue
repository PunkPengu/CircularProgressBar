<template>
  <div class="cpb__wrapper">
    <div class="cpb__percentage">{{ percentageText }}%</div>
    <svg class="cpb__progress" viewBox="0 0 150 150">
      <circle
        class="cpb__progress-background"
        :r="radius"
        cx="75"
        cy="75"
        :stroke-width="strokeWidth"
      />
      <circle
        class="cpb__progress-bar"
        :r="radius"
        cx="75"
        cy="75"
        :stroke-width="strokeWidth"
        stroke-linecap="round"
        :style="progressBarStyles"
      />
    </svg>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, watch } from "vue";

const props = withDefaults(
  defineProps<{
    progress: number;
    duration: number;
  }>(),
  {
    duration: 1000,
  }
);

// Settings
const radius = 60;
const strokeWidth = 30;

// Internal Settings
const radian = radius * 2 * Math.PI;

// Presets
const percentageText = ref(0);
const percentageValue = ref(0);

// Animation
const changePercentage = (val: number) => {
  const oldVal = percentageValue.value;
  percentageValue.value = val;

  const calcCurrentPercentage = (elapsedTime: number) => {
    return Math.floor(oldVal + (val - oldVal) * (elapsedTime / props.duration));
  };

  // Loop Function
  let startTime: number | undefined;
  const animationLoop = (stepTime: number) => {
    if (startTime === undefined) {
      startTime = stepTime;
    }
    const elapsedTime = stepTime - startTime;

    percentageText.value = calcCurrentPercentage(elapsedTime);

    // Loop animation
    if (elapsedTime < props.duration) {
      window.requestAnimationFrame(animationLoop);
    } else {
      percentageText.value = val;
    }
  };
  window.requestAnimationFrame(animationLoop);
};

// Animation Hooks
onMounted(() => {
  setTimeout(() => changePercentage(props.progress), 1);
});

watch(
  () => props.progress,
  (value) => {
    changePercentage(value);
  }
);

// Styles classes
const progressBarStyles = computed(() => {
  return {
    ["stroke-dasharray"]: `${radian}px ${radian}px`,
    ["stroke-dashoffset"]: `${
      radian - (percentageValue.value / 100) * radian
    }px`,
    ["transition"]: `stroke-dashoffset ${props.duration}ms ease-in-out`,
  };
});
</script>

<style lang="scss">
@import url("https://fonts.googleapis.com/css2?family=Fira+Sans:ital,wght@0,100;0,200;0,300;0,400;0,500;0,600;0,700;0,800;0,900;1,100;1,200;1,300;1,400;1,500;1,600;1,700;1,800;1,900&display=swap");

$clr-primary: #035370;
$clr-background: #ced4da80;
$fclr-primary: #333333;

* {
  box-sizing: border-box;
  font: normal normal bold 16px/22px "Fira Sans";
}

.cpb {
  &__wrapper {
    position: relative;
    width: 150px;
    height: 150px;
  }

  &__percentage {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    color: $fclr-primary;
  }

  &__progress {
    position: absolute;
    height: 100%;
    width: 100%;

    &-background {
      fill: transparent;
      stroke: $clr-background;
    }

    &-bar {
      fill: transparent;
      stroke: $clr-primary;
      transform: rotate(-90deg);
      transform-origin: 50% 50%;
    }
  }
}
</style>
