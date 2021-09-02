<template>
  <div class="scrambler-container">
    <h2 id="scrambled-word">{{ scrambledSentence }}</h2>
    <h3 class="explanatory-text">Guess the Sentence! Start typing</h3>
    <h3 class="explanatory-text">The yellow blocks are meant for the spaces</h3>
    <h3 class="score">Score: {{ score }}</h3>
    <!-- v-if directive used to conditionally display win message if player hits score of 10 -->
    <template v-if="score === 10">
      <div>
        <p class="win-msg">You Win!</p>
        <button @click="restart()" class="next-btn">Play Again</button>
      </div>
    </template>
    <!-- v-else: regular game guess boxes are displayed -->
    <template v-else>
      <form id="guess-form" class="guess-container">
        <template v-for="(letter, i) in checkedSentence" :key="i">
          <!-- index 'i" of v-for loop is used to assign ref and data-index values to inputs and button. used to focus navigate between them -->
          <template v-if="letter[0] == ' '">
            <input
              :class="{ green: letter.correct }"
              @input="checkLetter($event)"
              @keyup="focusNextField($event, 1)"
              @keyup.delete="focusPreviousField($event)"
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
            @keyup.delete="focusPreviousField($event)"
            class="guess-box"
            type="text"
            autofocus
            maxlength="1"
            :data-index="`${i}`"
            :ref="`letter-${i}`"
          />
        </template>
      </form>
      <!-- next button only displayed if showNextBtn evals to true. showNextBtn evals to true if input matches sentence. handled in checkLetter method -->
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
    // this computed property takes in the original sentence fetched from sentence api and scrambles based on the specifications listed in the instructions.
    scrambledSentence() {
      //sentence is split into an array by word
      const senArr = this.sentence.split(" ");
      let scrArr = [];
      // each word in array is checked for length. if 2 letter or less, it is pushed into the new scrambled array "scrArr"
      senArr.forEach((word) => {
        if (word.length <= 2) {
          scrArr.push(word);
        } else {
          // if word is more than 2 letters, it is split into a new array by letter. the first letter in the array is removed to maintain it in scramble
          const splitArr = word.split("");
          const sliceArr = splitArr.slice(1, splitArr.length - 1);
          // remaining letters are scrambled via the following for loop using th Fisher-Yates shuffle
          //
          const n = sliceArr.length;
          // i is set to n-1 to ignore the last letter in the array
          for (let i = n - 1; i > 0; i--) {
            const j = Math.floor(Math.random() * (i + 1));
            const tmp = sliceArr[i];
            sliceArr[i] = sliceArr[j];
            sliceArr[j] = tmp;
          }
          // first and last letters are added back and sentence is returned to string form with spaces
          sliceArr.unshift(splitArr[0]);
          sliceArr.push(splitArr[splitArr.length - 1]);
          scrArr.push(sliceArr.join(""));
        }
      });

      return scrArr.join(" ");
    },
    // original sentence split into array for use in other properties and methods
    splitSentence() {
      return this.sentence.split("");
    },
    // 'correct' boolean value added to each letter in splitSentence array to allow for each letter to be independently evaluated in v-for loop.
    addCorrectCheck() {
      const check = this.splitSentence.map((letter) => {
        return { ...letter, correct: false };
      });
      return check;
    },
  },
  methods: {
    // fetch sentence data from API
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
    // fetch next sentence from API in sequence
    // this method also serves to 'refresh' the page when next button is clicked or enter is pressed
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
      this.showNextBtn = false;
      // all inputs are cleared for next sentence
      document.getElementById("guess-form").reset();
      // inputSentence array which is used to check anser, is cleared for next sentence
      this.inputSentence = [];
      // first input of next sentence is focused
      this.$refs["letter-0"].focus();
    },
    // vue $refs are used to navigate between input fields
    focusPreviousField(event) {
      // 'previous element' is defined by the current elements data-set value -1 because inputs are in seqeuntial order in loop. example: the first input box of every sentence is ref: letter-0 and data-index: 0
      const previousEle =
        this.$refs[`letter-${Number(event.target.dataset.index) - 1}`];

      // checks if a previous element exists, and if the input of that element has eval'd as correct yet. This prevents player from deleting a letter that has eval'd as correct and messing up the push sequence to the inputSentence array and prevents errors if player trys to backspace when in the first input
      if (
        previousEle &&
        previousEle.value !== this.splitSentence[event.target.dataset.index - 1]
      ) {
        previousEle.focus();
        previousEle.value = null;
      }
    },
    // same as above. focus jumps to next input if value of input reaches max length of 1
    focusNextField(event, max) {
      if (event.key === "Backspace" || event.key === "Enter") {
        return event.preventDefault();
      }
      // if (event.target.value === "") {
      //   this.checkedSentence[event.target.dataset.index].correct = false;
      // }
      if (event.target.value.length === max) {
        const nextEle =
          this.$refs[`letter-${Number(event.target.dataset.index) + 1}`];
        if (nextEle) {
          nextEle.focus();
        }
      }
    },
    // method for verifying that input letter matches the letter in the original sentence
    // if letter matches, the correct boolean value of that letter is set to true and the letter is pushed in the input sentence array
    checkLetter(event) {
      if (
        event.target.value === this.splitSentence[event.target.dataset.index]
      ) {
        this.checkedSentence[event.target.dataset.index].correct = true;
        this.inputSentence.push(event.target.value);
      }
      // if inputSentence matches the original sentence, score is increased by one and the next button is revealed and focused
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
    // refreshes the page for the 'restart' button
    restart() {
      window.location.reload();
    },
  },
  mounted() {
    // initial data is fetched from api at mounted stage
    this.getData();
  },
  updated() {
    // checkedSentence variable set to new addCorrectCheck, adding in the 'correct' boolean value. to be used in v-for loop and other methods
    this.checkedSentence = this.addCorrectCheck;
  },
};
</script>
<style lang="scss" scoped>
// SCSS used for organization and scss variables declared in styles/base.scss
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