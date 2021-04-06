<template>
  <div class="flex justify-between">
    <SudokuBoard v-if="puzzleString === ''" :puzzle="origPuzzleString" />
    <SudokuBoard v-if="puzzleString !== ''" :puzzle="puzzleString" />

    <div class="w-64 h-64 m-3 steps_div">
      <h3 class="pb-2 text-center">Steps</h3>
      <div class="w-full h-full border border-black overflow-y-auto">
        <div v-if="stepsArr.length > 0">
          <div
            class="border-black border pl-2"
            :key="step"
            v-for="step in stepsArr"
          >
            {{ stepsArr.indexOf(step) + 1}}{{ ")" }} {{ step }}
          </div>
        </div>
      </div>
    </div>

    <div class="m-3">
      <div class="puzzleInput">
        <p class="pb-2 text-center">Puzzle Input:</p>
        <textarea
          type="text"
          v-model="origPuzzleString"
          name="origPuzzleString"
          placeholder="puzzle String here"
          class="w-64 h-40 border-black border-2 p-1"
        />
      </div>

      <button
        class="bg-gray-300 p-2 border-2 rounded-sm border-gray-400 border-double"
        @click="onSolve(origPuzzleString)"
      >
        Solve Puzzle
      </button>
    </div>
  </div>
</template>

<script>
import SudokuBoard from "./SudokuBoard";
export default {
  name: "SudokuSolver",
  components: {
    SudokuBoard
  },
  data() {
    return {
      origPuzzleString:
        "82..4..6...16..89...98315.749.157.............53..4...96.415..81..7632..3...28.51",
      puzzleString: "",
      stepsArr: []
    };
  },
  methods: {
    onSolve(newPuzzle) {
      // if puzzleString is complete then stop running
      if (!newPuzzle.includes(".")) {
        // console.log("solved");
        console.log("solution", newPuzzle);
        return newPuzzle;
      }

      // if not solved, begin solving by declaring variables
      this.puzzleString = newPuzzle;
      let cellsArr = [];
      let rowsArr = ["A", "B", "C", "D", "E", "F", "G", "H", "I"];
      let origPuzzle = newPuzzle;
      let newPuzzString = newPuzzle;
      // console.log("solving", newPuzzle);
      // console.log("puzzle state", this.puzzleString);

      // STEP 1: Find set of all possible solutions for each cell
      for (var z = 0; z < newPuzzle.length; z++) {
        // console.log("STEP 1 Entered");
        let rowNum = Math.floor(z / 9) + 1;
        let rowLetter = rowsArr[Math.floor(z / 9)];
        let colNum = (z % 9) + 1;
        let region = this.getRegion(rowLetter + colNum);
        var possSolArr = [];

        // only cycle through cell if it's not a number
        if (isNaN(newPuzzString[z])) {
          // cycle through 1-9 and push solutions to arrays for row, col, region
          for (var j = 1; j <= 9; j++) {
            var val = j.toString();
            if (this.checkPlacement(newPuzzString, rowLetter, colNum, val)) {
              possSolArr.push(j);
            }
          }

          /* create a new object with address and possible solutions */
          cellsArr.push(
            new this.CellObj(
              rowLetter + colNum,
              rowNum,
              colNum,
              z,
              region,
              possSolArr
            )
          );
        }
      }
      // console.log("cellsArr", cellsArr);

      // STEP 2: NAKED SINGLES, Insert all solutions from cells with one possible solution
      let singleSols = cellsArr.filter(obj => obj.possSolArr.length === 1);
      // console.log("singleSols", singleSols);
      if (singleSols.length > 0) {
        // console.log("STEP 2 Entered");
        let obj = singleSols[0];
        let loc = this.getLoc(obj.cellAddr);
        let val = obj.possSolArr[0];
        console.log("Naked Single added", val, "at", obj.cellAddr);
        this.stepsArr.push("Naked Single added " + val + " at " + obj.cellAddr);
        newPuzzString = this.addSolToStr(newPuzzString, loc, val);
      }

      /* If there was a change in STEP 2, rerun solve */
      if (newPuzzString !== origPuzzle) {
        this.puzzleString = newPuzzString;
        return this.onSolve(newPuzzString);
      }

      // STEP 3: Find HIDDEN SINGLES by row then by column
      // should run from x1 = 0 to x1 < 18 for all rows and columns
      for (var x1 = 0; x1 < 18; x1++) {
        // console.log("STEP 3 Entered");
        var filterVal = x1 < 9 ? rowsArr[x1] : x1 - 8;
        var rowOrCol = x1 < 9 ? "row" : "col";
        var rowSols = [];
        var rowA = cellsArr.filter(obj => obj[rowOrCol] == filterVal);
        rowA.forEach(obj => (rowSols = rowSols.concat(obj.possSolArr)));
        rowA.forEach(function(obj) {
          obj.possSolArr.forEach(function(val) {
            if (rowSols.indexOf(val) === rowSols.lastIndexOf(val)) {
              console.log(
                "Hidden Singles",
                rowOrCol,
                "added",
                val,
                "at",
                obj.cellAddr
              );
              this.stepsArr.push(
                "Hidden Singles " +
                  rowOrCol +
                  " added " +
                  val +
                  " at " +
                  obj.cellAddr
              );
              newPuzzString = this.addSolToStr(
                newPuzzString,
                obj.stringLoc,
                val
              );
            }
          }, this);
        }, this);
      }

      // /* If there was a change in STEP 3, rerun solve */
      if (newPuzzString !== origPuzzle) {
        this.puzzleString = newPuzzString;
        return this.onSolve(newPuzzString);
      }

      // STEP 4: Find REGION SINGLES, solutions unique in a region
      for (var reg = 1; reg <= 9; reg++) {
        // console.log("STEP 4 Entered");
        let arr = [];
        let regionSols = cellsArr.filter(obj => obj.region === reg);
        // console.log("regionSols", reg, regionSols);
        regionSols.forEach(obj => (arr = arr.concat(obj.possSolArr)));
        regionSols.forEach(obj =>
          obj.possSolArr.forEach(function(val) {
            if (arr.indexOf(val) === arr.lastIndexOf(val)) {
              console.log("Region Singles added", val, "at", obj.cellAddr);
              this.stepsArr.push(
                "Region Singles added " + val + " at " + obj.cellAddr
              );
              var loc = this.getLoc(obj.cellAddr);
              newPuzzString = this.addSolToStr(newPuzzString, loc, val);
            }
          }, this)
        );
      }
      /* If there was a change in STEP 4, rerun solve */
      if (newPuzzString !== origPuzzle) {
        this.puzzleString = newPuzzString;
        return this.onSolve(newPuzzString);
      }

      // // STEP 5: BRUTE FORCE with cells with two possible solutions
      if (newPuzzString === origPuzzle) {
        // console.log("STEP 5 entered");
        var twoSols = cellsArr.filter(sol => sol.possSolArr.length === 2);
        // console.log("Step 5 twoSol", twoSols);
        if (twoSols.length === 0) {
          console.log("no solution", origPuzzle);

          return false;
        }

        // console.log(twoSols)
        var newPuzzStringA = this.addSolToStr(
          newPuzzString,
          twoSols[0].stringLoc,
          twoSols[0].possSolArr[0]
        );
        // console.log("origPuzzle", origPuzzle);
        // console.log("newPuzzStringA", newPuzzStringA);
        var newPuzzStringB = this.addSolToStr(
          newPuzzString,
          twoSols[0].stringLoc,
          twoSols[0].possSolArr[1]
        );
        // console.log("newPuzzStringB", newPuzzStringB);
        console.log(
          "Brute Force added",
          twoSols[0].possSolArr[0],
          "at",
          twoSols[0].cellAddr
        );
        this.stepsArr.push(
          "Brute Force added" +
            twoSols[0].possSolArr[0] +
            " at " +
            twoSols[0].cellAddr
        );
        if (!this.onSolve(newPuzzStringA)) {
          console.log(
            "Brute Force added",
            twoSols[0].possSolArr[1],
            "at",
            twoSols[0].cellAddr
          );
          this.stepsArr.push(
            "Brute Force added " +
              twoSols[0].possSolArr[1] +
              " at " +
              twoSols[0].cellAddr
          );
          this.puzzleString = newPuzzStringB;
          return this.onSolve(newPuzzStringB);
        }
      } else {
        this.puzzleString = newPuzzString;
        return this.onSolve(newPuzzString);
      }
    },
    // object for each cell including address info and possible solutions for that cell
    CellObj(cellAddr, row, col, stringLoc, region, possSolArr) {
      this.cellAddr = cellAddr;
      this.row = row;
      this.col = col;
      this.stringLoc = stringLoc;
      this.region = region;
      this.possSolArr = possSolArr;
    },

    checkRowPlacement(puzzleString, row, column, value) {
      var rows = {
        A: 1,
        B: 2,
        C: 3,
        D: 4,
        E: 5,
        F: 6,
        G: 7,
        H: 8,
        I: 9
      };
      var rowArr = [];
      var puzzArr = [...puzzleString];
      var startPos = (rows[row] - 1) * 9;
      rowArr = puzzArr.slice(startPos, startPos + 9);
      return !rowArr.includes(value);
    },
    checkColPlacement(puzzleString, row, column, value) {
      var colArr = [];
      for (var i = 0; i < 9; i++) {
        colArr.push(puzzleString[i * 9 + (column - 1)]);
      }
      return !colArr.includes(value);
    },
    checkRegionPlacement(puzzleString, row, column, value) {
      var rows = {
        A: 1,
        B: 2,
        C: 3,
        D: 4,
        E: 5,
        F: 6,
        G: 7,
        H: 8,
        I: 9
      };
      var regionRow = rows[row] < 4 ? 1 : rows[row] < 7 ? 4 : 7;
      var regionCol = column < 4 ? 1 : column < 7 ? 4 : 7;
      var rowArr = puzzleString.slice((regionRow - 1) * 9, regionRow * 9 + 26);
      var regionArr = [];
      for (var i = 0; i < 3; i++) {
        var rowAdd = rowArr.slice(regionCol - 1 + i * 9, regionCol + 2 + i * 9);
        regionArr.push(rowAdd);
      }
      regionArr = regionArr.join("");

      return !regionArr.includes(value);
    },
    checkPlacement(puzzleString, row, column, value) {
      var rowTest = this.checkRowPlacement(puzzleString, row, column, value);
      var colTest = this.checkColPlacement(puzzleString, row, column, value);
      var regionTest = this.checkRegionPlacement(
        puzzleString,
        row,
        column,
        value
      );

      if (rowTest && colTest && regionTest) {
        return true;
      } else {
        return false;
      }
    },
    addSolToStr(string, index, replacement) {
      // console.log("addSolToStr", index, replacement);
      var tempSubstr1 = string.substr(0, index);
      var tempSubstr2 = string.substr(index + 1);
      var newString = tempSubstr1 + replacement + tempSubstr2;
      // console.log("newString", newString);
      return newString;
    },
    /* I added this to return location in puzzle string based on coordinate 'A1', 'A2', would return 0, 1, etc. */
    getLoc(string) {
      var rowsArr = ["A", "B", "C", "D", "E", "F", "G", "H", "I"];
      var loc = rowsArr.indexOf(string[0]) * 9 + (string[1] - 1);
      return loc;
    },
    getRegion(string) {
      let rowsArr = ["A", "B", "C", "D", "E", "F", "G", "H", "I"];
      let rowNum = rowsArr.indexOf(string[0]) + 1;
      let colNum = string[1];
      let region;
      /* Assign the region */
      if (rowNum < 4) {
        if (colNum < 4) {
          region = 1;
        } else if (colNum < 7) {
          region = 2;
        } else {
          region = 3;
        }
      } else if (rowNum < 7) {
        if (colNum < 4) {
          region = 4;
        } else if (colNum < 7) {
          region = 5;
        } else {
          region = 6;
        }
      } else {
        if (colNum < 4) {
          region = 7;
        } else if (colNum < 7) {
          region = 8;
        } else {
          region = 9;
        }
      }
      return region;
    },
    validate(puzzleString) {
      var puzzleObj = {
        regexTest: true,
        lengthTest: true
      };
      var regex = /^[0-9.]*$/;

      // Challenge #4, puzzle contains only digits and dots
      if (!regex.test(puzzleString)) {
        puzzleObj.regexTest = false;
      }

      // Challenge #5, puzzle length is 81
      if (puzzleString.length !== 81) {
        puzzleObj.lengthTest = false;
      }

      return puzzleObj;
    },
    // I added this test to test coordinate
    validateVal(val_input) {
      var regex = /^[1-9]$/;
      return regex.test(val_input);
    },
    // I added this test to test value
    validateCoord(coord_input) {
      var regex = /^[A-Ia-i][1-9]$/;
      return regex.test(coord_input);
    }
  }
};
</script>

<style>
textarea {
  resize: none;
}
</style>
