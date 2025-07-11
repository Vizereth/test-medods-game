<template>
  <main class="container">
    <p :class="['msg', { hidden: !gameOver }]">
      {{ `Игра окончена :(` }}
    </p>
    <div class="wrapper">
      <div :class="['difficulty', { hidden: isPlaying }]">
        <button
          @click="setDifficulty('easy')"
          :class="[
            'difficulty__btn',
            { 'difficulty__btn--selected': difficulty.name === 'easy' },
          ]"
          :disabled="isPlaying"
        >
          Легко
        </button>
        <button
          @click="setDifficulty('normal')"
          :class="[
            'difficulty__btn',
            { 'difficulty__btn--selected': difficulty.name === 'normal' },
          ]"
          :disabled="isPlaying"
        >
          Нормально
        </button>
        <button
          @click="setDifficulty('hard')"
          :class="[
            'difficulty__btn',
            { 'difficulty__btn--selected': difficulty.name === 'hard' },
          ]"
          :disabled="isPlaying"
        >
          Сложно
        </button>
      </div>
      <div class="stats">
        <p>Клик: {{ clicksTotal }}</p>
        <p>Раунд: {{ currentRound }}</p>
      </div>
      <div :class="['board', { locked: isLocked }]">
        <div v-for="(row, x) in matrix" :key="x" class="board__row">
          <div
            v-for="(_, y) in row"
            :key="`${x}-${y}`"
            :class="[
              `board__square board__square--${x}${y}`,
              {
                active: isTempSquare(x, y) && playingPattern,
              },
            ]"
            @click="handleSquareClick(x, y)"
          ></div>
        </div>
      </div>
    </div>
    <button
      :class="['btn', { hidden: isPlaying }]"
      :disabled="isPlaying"
      @click="startGame()"
    >
      Начать!
    </button>
  </main>
</template>

<script>
import { reactive, toRefs } from "@vue/composition-api";

import sound1 from "@/assets/sounds/1.mp3";
import sound2 from "@/assets/sounds/2.mp3";
import sound3 from "@/assets/sounds/3.mp3";
import sound4 from "@/assets/sounds/4.mp3";

const getRandomInt = (min, max) => {
  return Math.floor(Math.random() * (max - min + 1)) + min;
};

export default {
  setup() {
    const state = reactive({
      matrix: [
        [{ sound: new Audio(sound1) }, { sound: new Audio(sound2) }],
        [{ sound: new Audio(sound3) }, { sound: new Audio(sound4) }],
      ],
      difficultyConfig: {
        easy: { showSquareDelay: 1500, showSquareDuration: 500 },
        normal: { showSquareDelay: 1000, showSquareDuration: 500 },
        hard: { showSquareDelay: 400, showSquareDuration: 500 },
      },
      difficulty: {
        name: "easy",
        config: { showSquareDelay: 1500, showSquareDuration: 500 },
      },
      pattern: [],
      playingPattern: false,
      tempSquare: [],
      currentRound: 1,
      clicksTotal: 0,
      clicksNumber: 0,
      gameOver: false,
      isPlaying: false,
      isLocked: true,
    });

    function startGame() {
      reset();
      state.isPlaying = true;

      generatePattern();
      playPattern();
    }

    function endGame() {
      state.isPlaying = false;
      state.isLocked = true;
      state.gameOver = true;
    }

    function setDifficulty(k) {
      state.difficulty = { name: k, config: state.difficultyConfig[k] };
    }

    function reset() {
      state.pattern = [];
      state.tempSquare = [];
      state.currentRound = 1;
      state.gameOver = false;
      state.clicksNumber = 0;
      state.clicksTotal = 0;
    }

    function nextRound() {
      state.isLocked = true;

      state.clicksNumber = 0;
      state.currentRound += 1;
      state.pattern = [];

      generatePattern();
      playPattern();
    }

    function playPattern() {
      setTimeout(() => {
        state.playingPattern = true;

        state.pattern.forEach((square, index) => {
          setTimeout(() => {
            state.tempSquare = square;

            const [x, y] = square;
            state.matrix[x][y].sound.play();

            if (index === state.pattern.length - 1) {
              setTimeout(() => {
                state.playingPattern = false;
                state.isLocked = false;
              }, 1000);
            }
          }, (state.difficulty.config.showSquareDelay + state.difficulty.config.showSquareDuration) * index);
        });
      }, 1000);
    }

    function generatePattern() {
      function areCoordsEqual(coords1, coords2) {
        return (
          coords1 &&
          coords2 &&
          coords1[0] === coords2[0] &&
          coords1[1] === coords2[1]
        );
      }

      const newPattern = [];
      let lastCoords = null;

      for (let i = 0; i < state.currentRound; i++) {
        let newCoords;

        do {
          newCoords = generateCoords();
        } while (areCoordsEqual(newCoords, lastCoords));

        newPattern.push(newCoords);
        lastCoords = newCoords;
      }

      state.pattern = newPattern;
    }

    function generateCoords() {
      return [getRandomInt(0, 1), getRandomInt(0, 1)];
    }

    function handleSquareClick(x, y) {
      state.matrix[x][y].sound.play();

      const patternSquare = state.pattern[state.clicksNumber];
      const [patternSquareX, patternSquareY] = patternSquare;

      if (x === patternSquareX && y === patternSquareY) {
        state.clicksNumber += 1;
        state.clicksTotal += 1;
      } else {
        endGame();
        return;
      }

      if (state.clicksNumber === state.pattern.length) {
        nextRound();
      }
    }

    const isTempSquare = (x, y) => {
      return (
        state.tempSquare.length &&
        x === state.tempSquare[0] &&
        y === state.tempSquare[1]
      );
    };

    return {
      ...toRefs(state),
      generateCoords,
      generatePattern,
      playPattern,
      startGame,
      reset,
      handleSquareClick,
      isTempSquare,
      setDifficulty,
    };
  },
};
</script>

<style lang="scss">
// Colors
$color-1: #555;
$color-2: #666666;
$color-3: #ff4040;
$color-4: #ff9933;

$slice-color-1: #e74c3c; // vibrant red (like a juicy cherry)
$slice-color-2: #3498db; // bright, cool blue
$slice-color-3: #2ecc71; // fresh green, almost minty
$slice-color-4: #f1c40f;

// Other
$transition: 100ms ease-in;
$squareSize: 125px;
$device-mobile-small: 480px;

// Mixins
@mixin maxMobileSmall {
  @media screen and (max-width: $device-mobile-small) {
    @content;
  }
}

@keyframes showSlice {
  0% {
    filter: brightness(50%) grayscale(25%);
  }
  50% {
    filter: brightness(100%) grayscale(0%);
  }
  100% {
    filter: brightness(50%) grayscale(25%);
  }
}

.container {
  flex: 1 1 auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 50px;
  padding: 50px 0;

  &.locked {
    pointer-events: none;
  }
}

.wrapper {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.msg {
  color: $color-3;
  font-size: 18px;
}

.stats {
  display: flex;
  justify-content: space-between;
  gap: 10px;
  font-size: 14px;
  margin-bottom: 15px;
}

.difficulty {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 10px;

  margin-bottom: 25px;

  &__btn {
    padding: 10px 20px;
    transition: 0.25s;

    &--selected,
    &:active,
    &:focus {
      color: $color-4;
    }

    &:hover:not(.difficulty__btn--selected) {
      text-decoration: line-through;
    }
  }
}

.hidden {
  pointer-events: none;
  visibility: hidden;
}

.board {
  display: flex;
  flex-direction: column;
  gap: 10px;

  &.locked {
    pointer-events: none;
  }

  &__row {
    display: flex;
    gap: 10px;
  }

  &__square {
    filter: brightness(50%) grayscale(25%);
    width: $squareSize;
    height: $squareSize;

    &:hover {
      filter: brightness(100%) grayscale(0%);
      transform: scale(1.025);
    }

    &.active {
      animation: showSlice 400ms ease-out;
    }

    &--00 {
      background-color: $slice-color-1;
      border-top-left-radius: 100% 100%;
    }

    &--01 {
      background-color: $slice-color-2;
      border-top-right-radius: 100% 100%;
    }

    &--10 {
      background-color: $slice-color-3;
      border-bottom-left-radius: 100% 100%;
    }

    &--11 {
      background-color: $slice-color-4;
      border-bottom-right-radius: 100% 100%;
    }

    @include maxMobileSmall {
      width: $squareSize * 4/5;
      height: $squareSize * 4/5;
    }
  }
}

.btn {
  background: linear-gradient($color-1 0 0) 100% 0,
    linear-gradient($color-1 0 0) 0 0, linear-gradient($color-1 0 0) 0 100%,
    linear-gradient($color-1 0 0) 100% 100%;
  background-size: var(--d, 0) 1px, 1px var(--d, 0);
  background-repeat: no-repeat;
  transition: 0.25s;
  padding: 10px 20px;

  &:hover {
    --d: 100%;
  }
}
</style>
