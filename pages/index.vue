<template>
  <div>
    <h1 class="header bg-blue-200 text-center py-4">Peter's Sudoku Solver</h1>
    <div class="py-10 mx-24">
      <SudokuSolver class="sudokuSolver" />
      <InstructionsAndDef />
    </div>
  </div>
</template>

<script>
import SudokuSolver from "../components/SudokuSolver";
export default {
  components: {
    SudokuSolver
  }
};
</script>

<style>
/* Sample `apply` at-rules with Tailwind CSS
.container {
@apply min-h-screen flex justify-center items-center text-center mx-auto;
}
*/
@import url("https://fonts.googleapis.com/css2?family=Architects+Daughter&display=swap");
.container {
  margin: 0 auto;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: "Quicksand", "Source Sans Pro", -apple-system, BlinkMacSystemFont,
    "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
.header {
  font-family: "Architects Daughter";
  font-size: 26px;
  letter-spacing: 0.2em;
}
.sudokuSolver {
  font-family: "Architects Daughter";
  font-size: 18px;
}
</style>
