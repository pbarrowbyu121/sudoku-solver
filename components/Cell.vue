<template>
  <!-- <div class="col-span-1 border-black border-2 w-8 h-8"> -->
  <!-- <div class="cell border-black border-2"> -->
  <div :class="computedClass">
    {{ cellVal }}
  </div>
</template>

<script>
export default {
  name: "Cell",
  props: {
    cellVal: String,
    cellLoc: Number
  },
  computed: {
    computedClass() {
      let className = "cell border-black border text-center align-middle";
      this.cellLoc % 9 === 2 || this.cellLoc % 9 === 5
        ? (className = className + " border-r-4")
        : className;
      (this.cellLoc >= 18 && this.cellLoc <= 26) ||
      (this.cellLoc >= 45 && this.cellLoc <= 53)
        ? (className = className + " border-b-4")
        : className;
      return className;
    }
  }
};
</script>

<style scoped>
.cell {
  width: 2rem;
  height: 2rem;
  background: lightgray;
}
</style>
