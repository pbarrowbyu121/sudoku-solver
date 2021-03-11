<template>
  <div class="mt-6">
    <div class="instructions mt-6">
      <h2>Instructions:</h2>
      <ol>
        <li>
          1) Insert the puzzle string into the inputs box with blank cells
          represented with a "."
        </li>
        <li>
          2) Press the "Solve Puzzle" button
        </li>
        <li>3) Steps to solve may be viewed in the Steps box</li>
      </ol>
    </div>
    <div class="definitions mt-6">
      <h2>Definitions:</h2>
      <ol>
        <li>
          NAKED SINGLE: For a given cell, only one digit is possible
        </li>
        <li>
          HIDDEN SINGLES: For a given cell (which may have multiple possible
          candidates) one of the possible digits appears only in this cell as a
          possible digit in the same row and column
        </li>
        <li>
          REGION SINGLES: For a given cell (which may have multiple possible
          candidates), one of the possible digits appears only once in this cell
          for the corresponding region
        </li>
      </ol>
    </div>
  </div>
</template>

<script>
export default {
  name: "InstructionsAndDef"
};
</script>
