<template>
  <div class="scrambler-container">
    <h2 id="scrambled-word">{{ scrambledSentence }}</h2>
    <h3 class="explanatory-text">Guess the Sentence! Start typing</h3>
    <h3 class="explanatory-text">The yellow blocks are meant for the spaces</h3>
    <h3 class="score">Score: {{ score }}</h3>
    <template v-if="score === 10">
      <div>
        <p class="win-msg">You Win!</p>
        <button @click="restart()" class="next-btn">Play Again</button>
      </div>
    </template>
    <template v-else>
      <form id="guess-form" class="guess-container">
        <template v-for="(letter, i) in checkedSentence" :key="i">
          <template v-if="letter[0] == ' '">
            <input
              :class="{ green: letter.correct }"
              @input="checkLetter($event)"
              @keyup="focusNextField($event, 1)"
              @keyup.delete="previousInput($event)"
              class="guess-box space"
              type="text"
              maxlength="1"
              :data-index="`${i}`"
              :ref="`letter-${i}`"
            />
            <div class="break"></div>
          </template>
          <input
            v-else
            :class="{ green: letter.correct }"
            @input="checkLetter($event)"
            @keyup="focusNextField($event, 1)"
            @keyup.delete="previousInput($event)"
            class="guess-box"
            type="text"
            autofocus
            maxlength="1"
            :data-index="`${i}`"
            :ref="`letter-${i}`"
          />
        </template>
      </form>
      <button
        v-show="showNextBtn"
        class="next-btn"
        @click="getNewData(nextSentenceIndex)"
        @keyup.enter="getNewData(nextSentenceIndex)"
        :ref="`next-btn`"
      >
        Next
      </button>
    </template>
  </div>
</template>
<script>
import axios from "axios";

export default {
  name: "Scrambler",
  data() {
    return {
      sentenceIndex: 1,
      nextSentenceIndex: 2,
      data: {},
      sentence: "",
      score: 0,
      checkedSentence: [],
      inputSentence: [],
      showNextBtn: false,
    };
  },
  computed: {
    scrambledSentence() {
      const senArr = this.sentence.split(" ");
      let scrArr = [];
      senArr.forEach((word) => {
        if (word.length <= 2) {
          scrArr.push(word);
        } else {
          const splitArr = word.split("");
          const sliceArr = splitArr.slice(1, splitArr.length - 1);
          const n = sliceArr.length;
          for (let i = n - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            const tmp = sliceArr[i];
            sliceArr[i] = sliceArr[j];
            sliceArr[j] = tmp;
          }
          sliceArr.unshift(splitArr[0]);
          sliceArr.push(splitArr[splitArr.length - 1]);
          scrArr.push(sliceArr.join(""));
        }
      });

      return scrArr.join(" ");
    },
    splitSentence() {
      return this.sentence.split("");
    },
    addCorrectCheck() {
      const check = this.splitSentence.map((letter) => {
        return { ...letter, correct: false };
      });
      return check;
    },
  },
  watch: {},
  methods: {
    async getData() {
      let url = `https://api.hatchways.io/assessment/sentences/${this.sentenceIndex}`;
      try {
        const res = await axios.get(url);
        this.data = res.data.data;
        this.sentence = res.data.data.sentence.toLowerCase();
      } catch (err) {
        console.error(err);
      }
    },
    async getNewData(index) {
      let url = `https://api.hatchways.io/assessment/sentences/${index}`;
      try {
        const res = await axios.get(url);
        this.data = res.data.data;
        this.sentence = res.data.data.sentence.toLowerCase();
        this.nextSentenceIndex++;
      } catch (err) {
        console.error(err);
      }
      console.log(this.nextSentenceIndex);
      this.showNextBtn = false;
      document.getElementById("guess-form").reset();
      this.inputSentence = [];
      this.$refs["letter-0"].focus();
    },
    previousInput(event) {
      const previousEle =
        this.$refs[`letter-${Number(event.target.dataset.index) - 1}`];

      if (
        previousEle &&
        previousEle.value !== this.splitSentence[event.target.dataset.index - 1]
      ) {
        previousEle.focus();
        previousEle.value = null;
      }
    },
    focusNextField(event, max) {
      if (event.key === "Backspace" || event.key === "Enter") {
        return event.preventDefault();
      }
      if (event.target.value === "") {
        this.checkedSentence[event.target.dataset.index].correct = false;
      }
      if (event.target.value.length === max) {
        const nextEle =
          this.$refs[`letter-${Number(event.target.dataset.index) + 1}`];
        if (nextEle) {
          nextEle.focus();
        }
      }
    },
    checkLetter(event) {
      if (
        event.target.value === this.splitSentence[event.target.dataset.index]
      ) {
        this.checkedSentence[event.target.dataset.index].correct = true;
        this.inputSentence.push(event.target.value);
      }
      if (this.inputSentence.join("") === this.sentence) {
        this.score++;
        this.showNextBtn = true;
        if (this.score !== 10) {
          this.$nextTick(() => {
            this.$refs[`next-btn`].focus();
          });
        }
      }
    },
    restart() {
      window.location.reload();
    },
  },
  mounted() {
    this.getData();
  },
  updated() {
    this.checkedSentence = this.addCorrectCheck;
  },
};
</script>
<style lang="scss" scoped>
.scrambler-container {
  background: white;
  border-radius: 10px;

  margin: auto;
  margin-top: 2rem;
  padding: 0.5rem 1.5rem 1.5rem 1.5rem;

  min-height: 85vh;
  max-width: 80vw;

  font-family: $main-font;
  text-align: center;

  h2 {
    font-size: 2.2rem;
    color: $main-blue;
  }

  .score {
    font-size: 1.7rem;
  }

  .win-msg {
    font-size: 3.5rem;
  }

  .guess-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 1rem;
    padding: 1rem;

    .guess-box {
      background: $background-grey;
      min-height: 4rem;
      width: 1rem;
      display: block;
      flex-grow: 1;
      text-align: center;
      border: none;
      font-size: 3rem;
      caret-color: grey;

      &:focus {
        outline: none;
      }
    }
    .space {
      background-color: $space-yellow;
      width: 100%;
      height: 100%;
      flex-basis: 0%;

      flex-grow: 1;
    }
    .green {
      background-color: $correct-green;
    }

    .break {
      flex-basis: 100%;
      height: 0;

      flex-grow: 1;
    }
  }
  .next-btn {
    color: white;
    border: none;
    padding: 0.5rem 1rem;
    background: $next-green;
    font-size: 1rem;

    &:focus {
      outline: none;
    }
  }
}
</style>