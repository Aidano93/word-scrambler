<template>
  <div>
    <h2 id="scrambled-word">{{ sentence }}</h2>
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
      sentence: this.data.sentence
    };
  },
  computed: {
    scrambledSentence() {
      const senArr = this.sentence.split(" ");
      return {}
    }
  },
  async mounted() {
    let url = `https://api.hatchways.io/assessment/sentences/${this.fetchCounter}`;
    try {
      const res = await axios.get(url);
      this.data = res.data.data;
    } catch (err) {
      console.error(err);
    }
  },
};
</script>
<style lang="">
</style>