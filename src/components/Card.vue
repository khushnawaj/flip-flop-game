<template>
  <div class="card" @click="flipCard">
    <div class="card-inner" :class="{ flipped: card.flipped }">
      <div class="card-front">
        <p>{{ card.flipped ? card.value : '?'}}</p>
      </div>
      <div class="card-back">
        <p>{{ card.flipped ? "Flipped" : '? ' }}</p>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: ['card'],
  methods: {
    flipCard() {
      this.$emit('card-flip', this.card);
    },
  },
};
</script>

<style scoped>
.card {
  width: 100px;
  height: 150px;
  perspective: 1000px;
  margin: 10px;
}

.card-inner {
  width: 100%;
  height: 100%;
  transform-style: preserve-3d;
  transition: transform 0.6s;
}

.card-inner.flipped {
  transform: rotateY(180deg);
}

.card-front,
.card-back {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 20px;
  font-weight: bold;
}

.card-front {
  background: linear-gradient(#6edcd3, #d39c37);
  color: #420000;
  border-radius: 15px;

}

.card-back {
  background: linear-gradient(#4a5352, #5ed508);
  border: 1px solid green;
  border-radius: 15px;
    color: white;
    transform: rotateY(180deg);
}
</style>
