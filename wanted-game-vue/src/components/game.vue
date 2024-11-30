<template>
  <div class="game">
    <Controls @update:gameSettings="updateGameSettings" />

    <div
      v-for="(icon, index) in icons"
      :key="index"
      class="icon"
      :style="getIconStyle(icon)"
      @click="handleIconClick(icon)"
    >
      <img :src="icon.src" :alt="icon.alt" />
    </div>
  </div>
</template>

<script>
import Controls from './controls'
import jigglypuff from '@/assets/jigglypuff.png'
import pikachu from '@/assets/pikachu.png'
import snorlax from '@/assets/snorlax.png'

export default {
  name: 'WantedGame',
  components: {
    Controls,
  },
  data () {
    return {
      iconsArray: [
        { src: pikachu, alt: 'Pikachu', x: 0, y: 0, target: false, dx: 2, dy: 2 },
        { src: jigglypuff, alt: 'Jigglypuff', x: 100, y: 100, target: false,  dx: 3, dy: 3 },
        { src: snorlax, alt: 'Snorlax', x: 200, y: 200, target: true, dx: 2, dy: -2 }, // One is the correct target
      ],
      icons: [],
      gameSettings: {
        difficulty: 5,
        speed: 3,
      },
    }
  },
  mounted () {
    this.startIconMovement();
    this.generateIcons();
  },
  methods: {
    generateIcons() {
      for(let i = 0; i < this.gameSettings.difficulty; i++) {
        this.icons.push(this.iconsArray[0]);
        this.icons.push(this.iconsArray[1]);
      }
      this.icons.push(this.iconsArray[2]);
    },
    startIconMovement() {
      setInterval(() => {
        const container = document.querySelector('.game'); // Get game container
        const containerWidth = container.offsetWidth;
        const containerHeight = container.offsetHeight;

        this.icons.forEach(icon => {
          // Adjust speed based on difficulty and user input
          const speedFactor = 0.5;
          const dx = this.gameSettings.speed * speedFactor * (icon.dx > 0 ? 1 : -1);
          const dy = this.gameSettings.speed * speedFactor * (icon.dy > 0 ? 1 : -1);

          // Update the position of the icon
          icon.x += dx;
          icon.y += dy;

          // If the icon hits the right edge, reverse its horizontal direction
          if (icon.x >= containerWidth - 50 || icon.x <= 0) {
            icon.dx = -icon.dx + (Math.random() > 0.5 ? 1 : -1); // Randomize bounce direction
          }

          // If the icon hits the bottom edge, reverse its vertical direction
          if (icon.y >= containerHeight - 50 || icon.y <= 0) {
            icon.dy = -icon.dy + (Math.random() > 0.5 ? 1 : -1); // Randomize bounce direction
          }
        });
      }, 20)
    },
    // Get CSS styles for each icon
    getIconStyle(icon) {
      return {
        position: 'absolute',
        left: `${icon.x}px`,
        bottom: `${icon.y}px`,
      };
    },
    // Handle icon click (check if it's the correct one)
    handleIconClick(icon) {
      if (icon.target) {
        alert('You clicked the correct icon!');
      } else {
        alert('Wrong icon, try again!');
      }
    },
    updateGameSettings(newSettings) {
      this.gameSettings = { ...newSettings };
    },
  },
}
</script>

<style scoped>
.game {
  width: 100%;
  height: 100%;
}
.icon {
  width: 50px;
  height: 50px;
  cursor: pointer;
}
.icon img {
  width: 100%;
  height: 100%;
}
</style>