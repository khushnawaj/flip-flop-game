<template>
  <div id="app">
    <h1>Card Flip Game</h1>
    <div class="game-info">
      <div v-if="!gameStarted">
        <p>Select a difficulty to start the game:</p>
        <div class="difficulty-buttons">
          <button @click="startGame('easy')">Easy</button>
          <button @click="startGame('medium')">Medium</button>
          <button @click="startGame('hard')">Hard</button>
        </div>
      </div>
      <div v-else>
        <p>Moves: {{ moves }}</p>
        <p>Time: {{ timer }} seconds</p>
        <p v-if="gameOver" class="game-over-message">
          🎉 Game Over! You won in {{ timer }} seconds with {{ moves }} moves.
        </p>
        <div class="progress-bar">
          <div class="progress" :style="{ width: progressBarWidth + '%' }"></div>
        </div>
        <button @click="restartGame" class="restart-btn">Restart</button>
      </div>
    </div>
    <div class="card-container">
      <Card
        v-for="(card, index) in shuffledCards"
        :key="index"
        :card="card"
        :card-index="index"
        @card-flip="handleCardFlip"
      />
    </div>
  </div>
</template>

<script>
import Card from './components/Card.vue';

export default {
  components: {
    Card,
  },
  data() {
    return {
      shuffledCards: [],
      flippedCards: [],
      matchedCards: [],
      moves: 0,
      timer: 0,
      gameOver: false,
      gameStarted: false,
      difficulty: '',
      startTime: null,
      timerInterval: null,
      matchSound: null,
      flipSound: null,
      completeSound: null,
    };
  },
  computed: {
    progressBarWidth() {
      return (this.matchedCards.length / this.shuffledCards.length) * 100;
    },
  },
  created() {
    this.loadSounds();
  },
  methods: {
    loadSounds() {
      // Loading sounds for flip, match, and game complete
      this.flipSound = new Audio(require('./assets/flip.mp3'));
      this.matchSound = new Audio(require('./assets/match.mp3'));
      this.completeSound = new Audio(require('./assets/complete.mp3'));
    },
    startGame(difficulty) {
      this.difficulty = difficulty;
      this.shuffleAndGenerateCards(difficulty);
      this.startTimer();
      this.gameStarted = true;
    },
    restartGame() {
      this.gameStarted = false;
      this.gameOver = false;
      this.timer = 0;
      this.moves = 0;
      this.flippedCards = [];
      this.matchedCards = [];
      clearInterval(this.timerInterval);
    },
    shuffleAndGenerateCards(difficulty) {
      const cardPairs = {
        easy: 6,
        medium: 12,
        hard: 18,
      };
      const pairs = cardPairs[difficulty];
      const cardValues = Array.from({ length: pairs }, (_, i) => i + 1).flatMap(v => [v, v]);
      this.shuffledCards = this.shuffle(cardValues.map(value => ({ value, flipped: false })));
    },
    shuffle(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
      return array;
    },
    startTimer() {
      this.startTime = Date.now();
      this.timerInterval = setInterval(() => {
        if (!this.gameOver) {
          this.timer = Math.floor((Date.now() - this.startTime) / 1000);
        }
      }, 1000);
    },
    handleCardFlip(card) {
      // If the card is already flipped, or the game is over, do nothing
      if (this.flippedCards.length === 2 || card.flipped || this.gameOver) return;

      if (this.flipSound) {
        this.flipSound.play(); // Play flip sound
      }
      card.flipped = true;
      this.flippedCards.push(card);

      if (this.flippedCards.length === 2) {
        this.moves++;
        this.checkForMatch();
      }
    },
    checkForMatch() {
      const [firstCard, secondCard] = this.flippedCards;
      if (firstCard.value === secondCard.value) {
        this.matchedCards.push(firstCard, secondCard);
        this.flippedCards = [];
        if (this.matchSound) {
          this.matchSound.play(); // Play match sound
        }
        if (this.matchedCards.length === this.shuffledCards.length) {
          this.endGame();
        }
      } else {
        setTimeout(() => {
          firstCard.flipped = false;
          secondCard.flipped = false;
          this.flippedCards = [];
        }, 1000);
      }
    },
    endGame() {
      this.gameOver = true;
      clearInterval(this.timerInterval);
      if (this.completeSound) {
        this.completeSound.play(); // Play complete sound
      }
    },
  },
};
</script>

<style>
body {
  background: linear-gradient(45deg, #ff9a9e, #fad0c4);
  font-family: 'Poppins', sans-serif;
  margin: 0;
  color: #333;
}
#app {
  text-align: center;
  padding: 20px;
}
h1 {
  color: #fff;
  text-shadow: 2px 2px 8px rgba(0, 0, 0, 0.5);
}
.game-info {
  margin-bottom: 20px;
}
.difficulty-buttons button {
  margin: 10px;
  padding: 10px 20px;
  background: linear-gradient(45deg, #6a11cb, #2575fc);
  border: none;
  border-radius: 5px;
  color: white;
  cursor: pointer;
}
.difficulty-buttons button:hover {
  transform: scale(1.1);
  transition: 0.3s;
}
.card-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 15px;
}
.restart-btn {
  margin-top: 10px;
  padding: 10px 20px;
  background: linear-gradient(45deg, #ff6a00, #ee0979);
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
.progress-bar {
  width: 80%;
  height: 20px;
  background: #ddd;
  margin: 10px auto;
  border-radius: 10px;
  overflow: hidden;
}
.progress {
  height: 100%;
  background: linear-gradient(45deg, #00f260, #0575e6);
  transition: width 0.3s;
}
</style>
