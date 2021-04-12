<template>
  <div class="flex justify-between">
    <SudokuBoard v-if="puzzleString === ''" :puzzle="origPuzzleString" />
    <SudokuBoard v-if="puzzleString !== ''" :puzzle="puzzleString" />

    <div class="w-64 h-64 m-3 steps_div">
      <h3 class="pb-2 text-center">Steps</h3>
      <div id="steps-list" class="w-full h-full border border-black overflow-y-auto">
        <div v-if="stepsArr.length > 0">
          <div
            class="border-black border pl-2"
            :key="stepsArr.indexOf(step) + 1"
            v-for="step in stepsArr"
          >
            {{ stepsArr.indexOf(step) + 1 }}) {{ step.method}} {{ step.insertOrRemoved }} {{ step.val }} at {{ step.cellAddr }}
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
      <div class="puzzleInput">
        <p class="pb-2 text-center">Puzzle Solution:</p>
        <textarea
          type="text"
          v-model="puzzleString"
          name="puzzleString"
          placeholder="puzzle String here"
          class="w-64 h-40 border-black border-2 p-1"
        />
      </div>
      <div class="grid grid-cols-2" v-if="puzzleString.includes('.') || puzzleString === ''">
        <button
          class="bg-gray-300 p-2 border-2 rounded-sm border-gray-400 border-double"
          @click="onSolve(origPuzzleString)"
        >
          Solve Puzzle
        </button>
        <button
          class="bg-gray-300 p-2 border-2 rounded-sm border-gray-400 border-double"
          @click="solveStep(origPuzzleString)"
          v-if="stepsArr.length === 0"
        >
          Next Step 1
        </button>
        <button
          class="bg-gray-300 p-2 border-2 rounded-sm border-gray-400 border-double"
          @click="solveStep(puzzleString)"
          v-if="stepsArr.length > 0"
        >
          Next Step {{ stepsArr.length + 1}}
        </button>
      </div>
      <div v-if="!puzzleString.includes('.') && puzzleString !== ''" class="p-auto">SOLVED!</div>

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
        "697.....2..1972.63..3..679.912...6.737426.95.8657.9.241486932757.9.24..6..68.7..9",
      puzzleString: "",
      stepsArr: [],
      possSolutions: []
    };
  },
  methods: {
    // function to count occurences of an element in an array
    countOccurrences(arr, val) {return arr.reduce((a, v) => (v === val ? a + 1 : a), 0)},
    // function to get all possible solutions for all empty cells
    getPossSolutions(puzzleStringInput) {
      // if puzzle input is complete then no possible solutions
      if (!puzzleStringInput.includes(".")) {
        return false
      }

      // if not solved, begin solving by declaring variables
      let cellsArr = [];
      let rowsArr = ["A", "B", "C", "D", "E", "F", "G", "H", "I"];
      // let newPuzzString = newPuzzle;

      // Find set of all possible solutions for each cell
      for (var z = 0; z < puzzleStringInput.length; z++) {
        // console.log("STEP 1 Entered");
        let rowNum = Math.floor(z / 9) + 1;
        let rowLetter = rowsArr[Math.floor(z / 9)];
        let colNum = (z % 9) + 1;
        let region = this.getRegion(rowLetter + colNum);
        var possSolArr = [];

        // only cycle through cell if it's not a number
        if (isNaN(puzzleStringInput[z])) {
          // cycle through 1-9 and push solutions to arrays for row, col, region
          for (var j = 1; j <= 9; j++) {
            var val = j.toString();
            if (this.checkPlacement(puzzleStringInput, rowLetter, colNum, val)) {
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
      return cellsArr
    },
    getNakedSingles(puzzleStringInput) {
      // NAKED SINGLES, Insert all solutions from cells with one possible solution
      let singleSols = this.possSolutions.filter(cell => cell.possSolArr.length === 1);
      // console.log("singleSols", singleSols);
      if (singleSols.length > 0) {
        // console.log("STEP 2 Entered");
        let cellObj = singleSols[0];
        let loc = this.getLoc(cellObj.cellAddr);
        let val = cellObj.possSolArr[0];
        let newStepObj = {
          // step: this.stepsArr.length + 1,
          method: "Naked Single",
          cellAddr: cellObj.cellAddr,
          insertOrRemoved: "added",
          val: val
        }
        this.stepsArr = [...this.stepsArr, newStepObj]
        // this.stepsArr.push("Naked Single added " + val + " at " + obj.cellAddr);
        let newPuzzString = this.addSolToStr(puzzleStringInput, loc, val);
        this.puzzleString = newPuzzString
        console.log(newStepObj.step + ":", newStepObj.method + " added ", newStepObj.val, "at ", newStepObj.cellAddr)
        console.log(newStepObj.method + " added ", newStepObj.val, "at ", newStepObj.cellAddr)
        return true
      } else {
        return false
      }
    },
    getHiddenSingles(puzzleStringInput) {
      // Find HIDDEN SINGLES by row then by column
      let rowsArr = ["A", "B", "C", "D", "E", "F", "G", "H", "I"];
      // should run from x1 = 1 to x1 <= 18, 1-9 for all rows, 10-18 for all cols
      for (let x1 = 1; x1 <= 18; x1++) {
        let filterVal = x1 <= 9 ? x1 : x1 - 9;
        let rowOrCol = x1 <= 9 ? "row" : "col";
        let rowSols = [];
        // filter cellsArr for cells in the same row or column
        let rowA = this.possSolutions.filter(cellObj => cellObj[rowOrCol] == filterVal);
        // take all the solutions for a given row or column and put them in one array
        rowA.forEach(cellObj => (rowSols = rowSols.concat(cellObj.possSolArr)));
        console.log(rowOrCol, filterVal, "solutions: ", rowSols)
        // cycle through each possible solution. If it only appears once in the row or column then it's a HIDDEN SINGLE
        for (let rowColCellIndex = 0; rowColCellIndex < rowA.length; rowColCellIndex++) {
          for (let possSolIndex = 0; possSolIndex < rowA[rowColCellIndex].possSolArr.length; possSolIndex++) {
            let testCell = rowA[rowColCellIndex]
            let testSol = testCell.possSolArr[possSolIndex]
            if (rowSols.indexOf(testSol) === rowSols.lastIndexOf(testSol)) {
              let newStepObj = {
                // step: this.stepsArr.length + 1,
                method: "Hidden Singles",
                cellAddr: testCell.cellAddr,
                insertOrRemoved: "added", 
                val: testSol
              }
              this.stepsArr = [...this.stepsArr, newStepObj]
              let newPuzzString = this.addSolToStr(
                puzzleStringInput,
                testCell.stringLoc,
                testSol
              )
              this.puzzleString = newPuzzString
              // console.log(newStepObj.step + ":", newStepObj.method + " added ", newStepObj.val, "at ", newStepObj.cellAddr)
              console.log(newStepObj.method + " added ", newStepObj.val, "at ", newStepObj.cellAddr)
              return true
            }
          }
        }
      }
      return false
    },
    getRegionSingles(puzzleStringInput) {
      // Find REGION SINGLES, solutions unique in a region
      // cycle through each region
      for (let reg = 1; reg <= 9; reg++) {
        let arr = [];
        // filter cellsArr for cells in the region to get possible solutions for this region
        let regionSols = this.possSolutions.filter(cellObj => cellObj.region === reg);
        // create single array of all possible solutions for each cell in this region
        regionSols.forEach(cellObj => (arr = arr.concat(cellObj.possSolArr)));
        for (let regionSolIndex = 0; regionSolIndex < regionSols.length; regionSolIndex++) {
          for (let possSolIndex = 0; possSolIndex < regionSols[regionSolIndex].possSolArr.length; possSolIndex++) {
            // if solution appears only once in regions solutions array then it's a region single
            let testVal = regionSols[regionSolIndex].possSolArr[possSolIndex]
            if (arr.indexOf(testVal) === arr.lastIndexOf(testVal)) {
              let newStepObj = {
                // step: this.stepsArr.length + 1,
                method: "Region Single",
                insertOrRemoved: "added",
                cellAddr: regionSols[regionSolIndex].cellAddr,
                val: testVal
              }
              this.stepsArr = [...this.stepsArr, newStepObj]
              var loc = this.getLoc(regionSols[regionSolIndex].cellAddr);
              this.puzzleString = this.addSolToStr(puzzleStringInput, loc, testVal);
              // console.log(newStepObj.step + ":", newStepObj.method + " added ", newStepObj.val, "at ", newStepObj.cellAddr)
              console.log(newStepObj.method + " added ", newStepObj.val, "at ", newStepObj.cellAddr)
              return true
            }
          }
        }
      }
      return false
    },
    skyscraperPatternRoc(rowOrCol) {
      // Find SKYSCRAPERS by row
      // Focus on a digit. Find two rows (or columns) where that digit occurs in possible solutions twice in each row (or column)
      // Determine if the two of those cells with the digit are in the same column (or row). Call this column (or row) the "connection"
      // Eliminate that digit from cells in the same column as remaining cells in the original two rows (or columns) but not in the "connection" column (or row)

      // focus on a digit 1-9 at a time
      for (let testDigit = 1; testDigit <= 9; testDigit++) {
        // create an object to store possible row candidates where digit appears 2x
        let rocCandidatesForDigit = {
          digit: testDigit,
          candidateRocs: [] 
        }
        let skyScrapingPairs = []
        // cycle through each row and find rows where test digit occurs twice
        // should run from x1 = 1 to x1 <= 18, 1-9 for all rows, 10-18 for all cols
        for (let x1 = 1; x1 <=9; x1++) {
          let filterVal = x1 <= 9 ? x1 : x1 - 9;
          // let rowOrCol = x1 <= 9 ? "row" : "col";
          let rocSols = [];
          // let rocA = []
          let candidateRocsObj = {
              roc: x1,
              cors: []
          }
          
          // filter possSolutions for cells in the x1 row (or column)
          let rocA = rowOrCol === "row" ? this.possSolutions.filter(cellObj => cellObj.row == filterVal) : this.possSolutions.filter(cellObj => cellObj.col == filterVal);
          // take all the solutions for a given row or column and put them in one array to see if testDigit occurs 2x
          rocA.forEach(cellObj => cellObj.possSolArr.forEach(possSol => {
            rocSols.push(possSol)
            // rowSolsAddr.push(cellObj.cellAddr)
          }))
  
          // see if testDigit occurs two times in possible solutions for a given row (or column)
          if (this.countOccurrences(rocSols, testDigit) == 2) {
            // if testDigit does occur twice, get the cells in this row that have that digit, put in cellsWithDigit array
            let cellsWithDigit = rowOrCol === "row" ? this.possSolutions.filter(cellObj => cellObj.possSolArr.includes(testDigit) && cellObj.row === filterVal) : this.possSolutions.filter(cellObj => cellObj.possSolArr.includes(testDigit) && cellObj.col === filterVal)
            
            candidateRocsObj.cors = rowOrCol === "row" ? [cellsWithDigit[0].col, cellsWithDigit[1].col] : [cellsWithDigit[0].row, cellsWithDigit[1].row]
            // console.log("row", x1, "sees 4 twice in column", cellsWithDigit[0].col, "and column", cellsWithDigit[1].col)
            rocCandidatesForDigit.candidateRocs.push(candidateRocsObj)
          }
        }
        
        // for each row that is a candidate (with two occurrences of digit), make list of all possible combos (size 2) with other candidate rows      
        let result = rocCandidatesForDigit.candidateRocs.reduce( (acc, v, i) =>
          acc.concat(rocCandidatesForDigit.candidateRocs.slice(i+1).map( w => [v, w] )),
          []);

        // compare rows to see if they have columns in common where digit occurs, call that column 'connection'
        for (let i = 0; i < result.length; i++) {
          let pair = result[i]
          let testArr = [...pair[0].cors, ...pair[1].cors]
          let uniqueChars = [...new Set(testArr)]
          
          // if uniqueChars is less than 4 that means one of the columns is common between the two rows and thus a solution can be removed
          if (uniqueChars.length < 4) {
            if(pair[1].cors.includes(pair[0].cors[0]) ) {
              pair.connection = pair[0].cors[0]
            } else {
              pair.connection = pair[0].cors[1]
            }

            skyScrapingPairs.push(pair)

            // get cells that are not in the connection cor but have this digit
            // create array of cells to be altered
            let alterArr = []
            if(rowOrCol === "row") {
              alterArr = this.possSolutions.filter(obj => testArr.includes(obj.col) && obj.col !== pair.connection && obj.possSolArr.includes(testDigit) && ![pair[0].roc, pair[1].roc].includes(obj.row)) 
            } else {
              alterArr = this.possSolutions.filter(obj => testArr.includes(obj.row) && obj.row !== pair.connection && obj.possSolArr.includes(testDigit) && ![pair[0].roc, pair[1].roc].includes(obj.col)) 
            }
            
            // if the array of cells to be altered is not empty
            if (alterArr.length > 0) {
              
              let keepArr = this.possSolutions.filter(obj => !alterArr.includes(obj))
              let alteredCells = []
              alterArr.forEach(obj => {
                alteredCells = [...alteredCells, obj.cellAddr]
                obj.possSolArr = obj.possSolArr.filter(sol => sol !== 4)
              })
              let newStepObj = {
                  // step: this.stepsArr.length + 1,
                  method: "Skyscraping " + rowOrCol,
                  insertOrRemoved: "removed",
                  cellAddr: alteredCells.join(', '),
                  val: testDigit
              }
              if(!this.arrayIncludesObject(this.stepsArr, newStepObj)) {
                this.stepsArr.push(newStepObj)
                this.possSolutions = [...keepArr, ...alterArr]
                if (this.puzzleString === '') {
                  this.puzzleString = this.origPuzzleString
                }
                // console.log("alterArr after", alterArr)
                // console.log("keepArr", keepArr)
                console.log("SKYSCRAPING", rowOrCol, "removed", testDigit, "from", newStepObj.cellAddr)
                return true
              }
            }
          }
        }
      }
      return false
    },
    // function to compare two objects (keys must be in same order)
    compareObjects(obj1, obj2) {
      if (Object.keys(obj1).toString() === Object.keys(obj2).toString() && Object.values(obj1).toString() === Object.values(obj2).toString()) {
        return true
      } else { return false }
    },
    arrayIncludesObject(arr, obj) {
      return arr.filter(object => this.compareObjects({...object}, obj)).length > 0 ? true : false
    },
    solveStep(newPuzzle) {
      // if puzzleString is complete then stop running
      if (!newPuzzle.includes(".")) {
        // console.log("solved");
        console.log("solution", newPuzzle);
        return false;
      }

      // STEP 1: Find set of all possible solutions for each cell
      this.possSolutions = this.getPossSolutions(newPuzzle)

      // // STEP 2: NAKED SINGLES, Insert all solutions from cells with one possible solution
      // if (this.getNakedSingles(newPuzzle)) {
      //   return true
      // } else {
      //   console.log("Step", (this.stepsArr.length + 1) + ": NAKED SINGLES yielded no results")
      // }

      // STEP 3: Find HIDDEN SINGLES by row then by column
      // if (this.getHiddenSingles(newPuzzle)) {
      //   return true
      // } else {
      //   console.log("Step", (this.stepsArr.length + 1) + ": HIDDEN SINGLES yielded no results")
      // }

      // // STEP 4: Find REGION SINGLES, solutions unique in a region
      // if (this.getRegionSingles(newPuzzle)) {
      //   return true
      // } else {
      //   console.log("Step", (this.stepsArr.length + 1) + ": REGION SINGLES yielded no results")
      // }

      // METHOD 5: Find SKYSCRAPER patterns to eliminate some solutions
      if (this.skyscraperPatternRoc("row")) {
        return true
      } else {
        console.log("Step", (this.stepsArr.length + 1) + ": SKYSCRAPER PATTERN ROWS yielded no results")
      }

      if (this.skyscraperPatternRoc("col")) {
        return true
      } else {
        console.log("Step", (this.stepsArr.length + 1) + ": SKYSCRAPER PATTERN COLS yielded no results")
      }

      // this.scrollToEnd()
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
    },
    scrollToEnd() {
      let stepsList = this.$el.querySelector("#steps-list");
      
      stepsList.scrollTop = stepsList.scrollHeight;
    }
  }
};
</script>

<style>
textarea {
  resize: none;
}
</style>
