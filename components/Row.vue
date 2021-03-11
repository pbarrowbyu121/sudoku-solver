<template>
  <div></div>
</template>

<script>
import Cell from "./Cell";
export default {
  name: "Row",
  components: {
    Cell
  },
  data() {
    return {
      columns: [0, 1, 2, 3, 4, 5, 6, 7, 8]
    };
  }
};
</script>
