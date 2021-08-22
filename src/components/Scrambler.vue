<template>
  <div class="scrambler-container">
    <h2 id="scrambled-word">{{ scrambledSentence }}</h2>
    <h3 class="explanatory-text">Guess the Sentence! Start typing</h3>
    <h3 class="explanatory-text">The yellow blocks are meant for the spaces</h3>
    <h3 class="score">Score: {{ score }}</h3>
    <div class="guess-container">
      <template v-for="(letter, i) in splitSentence" :key="i">
        <template v-if="letter == ' '">
          <div class="guess-box space"></div>
          <div class="break"></div>
        </template>
        <div v-else class="guess-box"></div>
      </template>
    </div>
    <button class="next-btn" @click="getNewData(nextSentenceIndex)">
      Next
    </button>
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
  },
  watch: {},
  methods: {
    async getData() {
      let url = `https://api.hatchways.io/assessment/sentences/${this.sentenceIndex}`;
      try {
        const res = await axios.get(url);
        this.data = res.data.data;
        this.sentence = res.data.data.sentence;
      } catch (err) {
        console.error(err);
      }
    },
    async getNewData(index) {
      let url = `https://api.hatchways.io/assessment/sentences/${index}`;
      try {
        const res = await axios.get(url);
        this.data = res.data.data;
        this.sentence = res.data.data.sentence;
        this.nextSentenceIndex++;
      } catch (err) {
        console.error(err);
      }
    },
    getNextSentence() {
      this.sentenceIndex++;
      console.log(this.sentenceIndex);
    },
  },
  mounted() {
    this.getData();
  },
};
</script>
<style lang="scss" scoped>
.scrambler-container {
  background: white;
  border-radius: 10px;

  margin: auto;
  margin-top: 2.5rem;
  padding-top: 1rem;

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

  .guess-container {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    gap: 1rem;
    padding: 1rem;

    .guess-box {
      background: $background-grey;
      min-height: 4rem;

      flex-grow: 1;
    }
    .space {
      background-color: $space-yellow;
      width: 100%;
      height: 100%;
      flex-basis: 0%;

      flex-grow: 1;
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
  }
}
</style>