<template>
  <div class="c-topBar" v-if="currentStep && steps">
    <div>
      <strong>{{ selectedStep.name }}</strong>
    </div>
    <div>Krok: {{ currentStep }} z {{ steps.length }}</div>
  </div>
</template>
<script>
export default {
  props: {
    currentStep: {
      default: () => 1,
    },
    steps: {
      default: () => [],
    },
  },
  computed: {
    selectedStep() {
      return this.steps[this.currentStep - 1];
    },
  },
};
</script>
<style lang="scss" scoped>
.c-topBar {
  position: sticky;
  top: 0;
  left: 0;
  right: 0;
  background-color: #ffffff;
  color: #333;
  padding: 15px 40px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  z-index: 2000;
  border-bottom: 1px solid #eee;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.02);

  strong {
    font-size: 1rem;
    color: #d9534f;
    font-weight: 700;
  }

  div:last-child {
    font-size: 0.85rem;
    color: #666;
    background: #f1f3f5;
    padding: 4px 12px;
    border-radius: 20px;
  }
}
</style>
