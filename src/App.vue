<template>
  <Scrambler />
</template>

<script>
import Scrambler from "./components/Scrambler.vue";

export default {
  name: "App",
  components: {
    Scrambler,
  },
};
</script>

<style lang="scss">
body {
  background: $background-grey;
}
</style>
