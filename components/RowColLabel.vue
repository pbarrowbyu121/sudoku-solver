<template>
  <p class="cell bg-gray-100 text-center align-middle">
    {{ cellVal }}
  </p>
</template>

<script>
export default {
  name: "RowColLabel",
  props: {
    cellVal: String,
    cellLoc: Number
  }
};
</script>

<style scoped>
.cell {
  width: 2rem;
  height: 2rem;
}
</style>
