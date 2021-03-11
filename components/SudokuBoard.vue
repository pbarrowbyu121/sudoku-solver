<template>
  <!-- <div class="h-72 grid grid-flow-row grid-cols-9 border-black border-2"> -->
  <div class="sudokuBoardCanvas m-3">
    <!-- <h2>Sudoku Board</h2> -->
    <div class="row_labels">
      <RowColLabel :cellVal="rows[n - 1]" :key="n - 1" v-for="n in 9" />
    </div>
    <div class="colLabel_and_board">
      <div class="col_labels">
        <RowColLabel :cellVal="columns[n - 1]" :key="n - 1" v-for="n in 9" />
      </div>
      <div class="sudokuBoard">
        <Cell
          :cellVal="puzzle[n - 1]"
          :cellLoc="n - 1"
          :key="n - 1"
          v-for="n in 81"
        />
      </div>
    </div>
  </div>
</template>

<script>
import Cell from "./Cell";
import RowColLabel from "./RowColLabel";
export default {
  name: "SudokuBoard",
  components: {
    Cell,
    RowColLabel
  },
  props: {
    puzzle: String
  },
  data() {
    return {
      rows: ["A", "B", "C", "D", "E", "F", "G", "H", "I"],
      columns: ["1", "2", "3", "4", "5", "6", "7", "8", "9"]
    };
  }
};
</script>

<style scoped>
.sudokuBoard,
.col_labels {
  display: grid;
  grid-template-columns: repeat(9, minmax(0, 2rem));
  grid-gap: 0px;
  grid-auto-flow: row;
  width: auto;
  justify-content: center;
}
.sudokuBoardCanvas {
  display: grid;
  grid-template-columns: 2rem 18rem;
  grid-gap: 0px;
  grid-auto-flow: row;
  width: auto;
}
.row_labels {
  padding-top: 2rem;
}
</style>
