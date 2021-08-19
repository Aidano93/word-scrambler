<template>
  <div>
    <h2 id="scrambled-word">{{ scrambledSentence }}</h2>
  </div>
</template>
<script>
import axios from "axios";

export default {
  name: "Scrambler",
  data() {
    return {
      fetchCounter: 1,
      data: {},
      sentence: "",
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
  },
  async mounted() {
    let url = `https://api.hatchways.io/assessment/sentences/${this.fetchCounter}`;
    try {
      const res = await axios.get(url);
      this.data = res.data.data;
      this.sentence = res.data.data.sentence;
    } catch (err) {
      console.error(err);
    }
  },
};
</script>
<style lang="">
</style>