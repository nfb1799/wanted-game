<template>
  <div class="controls-container">
    <div class="control">
      <label for="difficulty">Difficulty</label>
      <input
        type="range"
        id="difficulty"
        min="1"
        max="10"
        v-model="difficulty"
        @input="updateControls"
      />
      <span>{{ difficulty }}</span>
    </div>

    <div class="control">
      <label for="speed">Speed</label>
      <input
        type="range"
        id="speed"
        min="1"
        max="5"
        v-model="speed"
        @input="updateControls"
      />
      <span>{{ speed }}</span>
    </div>
  </div>
</template>

<script>
export default {
  name: 'controlsComponent',
  data() {
    return {
      // Default difficulty (affects the speed of icons)
      difficulty: 5,
      // Default speed for the icons' movement
      speed: 3,
    };
  },
  methods: {
    // Emit the control values to the parent component
    updateControls() {
      this.$emit('update:gameSettings', {
        difficulty: this.difficulty,
        speed: this.speed,
      });
    },
  },
};
</script>

<style scoped>
.controls-container {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  padding: 20px;
  background-color: #f0f0f0;
  border-radius: 8px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
  width: 250px;
}

.control {
  margin-bottom: 15px;
  display: flex;
  align-items: center;
}

.control label {
  margin-right: 10px;
  font-size: 14px;
}

.control input[type="range"] {
  width: 100px;
}

.control span {
  margin-left: 10px;
  font-size: 16px;
}
</style>
