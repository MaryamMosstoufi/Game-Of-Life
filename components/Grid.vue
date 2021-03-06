<template>
  <div>
    <Stats
      :current-generation="currentGeneration"
      :cell-count="cellCount"
      :cells-alive="cellsAlive"
      :cells-death="cellsDeath"
      :cells-created="cellsCreated"
      :current-speed="currentSpeed"/>
    <div
      class="game-grid"
      @mousedown="isMouseDown = true"
      @mouseup="isMouseDown = false"
      @mouseleave="isMouseDown = false">
      <div
        v-for="(col, indexX) in gridList"
        :key="indexX"
        class="game-column">
        <Cell
          v-for="(isAlive, indexY) in col"
          :key="indexY"
          :status-obj="isAlive"
          :x-pos="indexX"
          :y-pos="indexY"
          :is-mouse-down="isMouseDown"
          @wasUpdated="updateCellCount"
        />
      </div>
    </div>
  </div>
</template>

<script>
import Cell from "./Cell.vue";
import Stats from "./Stats.vue";
export default {
  components: {
    Cell,
    Stats,
  },
  props: {
    message: {
      default: "",
      type: String,
    },
    importToken: {
      default: "",
      type: String,
    },
    currentSpeed: {
      default: 0,
      type: Number,
    },
  },
  data() {
    return {
      width: 100,
      height: 40,
      gridList: [],
      currentGeneration: 0,
      cellCount: 0,
      cellsAlive: 0,
      cellsDeath: 0,
      cellsCreated: 0,
      isMouseDown: false,
    };
  },
  computed: {},
  watch: {
    message: function (val) {
      if (val === "nextStep") {
        this.update();
        this.currentGeneration++;
      } else if (val === "redoSession") {
        this.reset();
      } else if (val === "randomSeed") {
        this.randomSeed();
      } else if (val === "importSession") {
        this.importSession();
      } else if (val === "exportSession") {
        this.exportSession();
      }
    },
  },
  created() {
    this.cellCalc();
  },
  methods: {
    cellCalc: function () {
      for (let i = 0; i < this.width; i++) {
        this.gridList[i] = [];
        for (let j = 0; j < this.height; j++) {
          this.gridList[i][j] = { isAlive: false };
        }
      }
      this.cellCount = this.width * this.height;
    },
    setCell: function (x, y, bool) {
      if (this.gridList[x][y].isAlive != bool) {
        this.gridList[x][y].isAlive = bool;
        this.updateCellCount(bool);
      }
    },
    update: function () {
      let tempArr = [];
      for (let i = 0; i < this.width; i++) {
        tempArr[i] = [];
        for (let j = 0; j < this.height; j++) {
          let status = this.gridList[i][j].isAlive;
          let neighbours = this.getNeighbours(i, j);
          let result = false;
          // Rule 1:
          // Any live cell with fewer than two live neighbours dies, as if by under population.
          if (status && neighbours < 2) {
            result = false;
          }
          // Rule 2:
          // Any live cell with two or three neighbours lives on to the next generation.
          if ((status && neighbours == 2) || neighbours == 3) {
            result = true;
          }
          // Rule 3:
          // Any live cell with more than three live neighbours dies, as if by overpopulation.
          if (status && neighbours > 3) {
            result = false;
          }
          // Rule 4:
          // Any dead cell with exactly three live neighbours becomes a live cell, as if by reproduction.
          if (!status && neighbours == 3) {
            result = true;
          }
          tempArr[i][j] = result;
        }
      }
      
      for (let i = 0; i < this.width; i++) {
        for (let j = 0; j < this.height; j++) {
          this.setCell(i, j, tempArr[i][j]);
        }
      }
    },
    getNeighbours: function (posX, posY) {
      let neighbours = 0;
      if (posX <= this.width && posY <= this.height) {
        for (let offsetX = -1; offsetX < 2; offsetX++) {
          for (let offsetY = -1; offsetY < 2; offsetY++) {
            let newX = posX + offsetX;
            let newY = posY + offsetY;
            
            if (
              (offsetX != 0 || offsetY != 0) &&
              newX >= 0 &&
              newX < this.width &&
              newY >= 0 &&
              newY < this.height &&
              this.gridList[posX + offsetX][posY + offsetY].isAlive == true
            ) {
              neighbours++;
            }
          }
        }
      }
      return neighbours;
    },
    reset: function () {
      this.currentGeneration = 0;
      this.cellsAlive = 0;
      this.cellsDeath = 0;
      this.cellsCreated = 0;
      this.gridList.forEach((col) => {
        col.forEach((cell) => {
          cell.isAlive = false;
        });
      });
    },
    randomSeed: function () {
      for (let i = 0; i < this.width; i++) {
        for (let j = 0; j < this.height; j++) {
          let rand = Math.random();
          if (rand < 0.2) {
            this.setCell(i, j, true);
          } else {
            this.setCell(i, j, false);
          }
        }
      }
    },
    importSession: function () {
      this.reset();
      let regex = /\[\d+,\d+\]/gm;
      let tempArr = this.importToken.match(regex);
      if (tempArr) {
        tempArr.forEach((element) => {
          element = element.substring(1, element.length - 1);
          let xy = element.split(",");
          this.setCell(xy[0], xy[1], true);
        });
      }
    },
    exportSession: function () {
      let exportToken = "";
      for (let i = 0; i < this.width; i++) {
        for (let j = 0; j < this.height; j++) {
          if (this.gridList[i][j].isAlive) {
            exportToken += "[" + i + "," + j + "]";
          }
        }
      }
      this.$emit("exportToken", exportToken);
    },
    updateCellCount: function (bool) {
      if (bool) {
        this.cellsAlive++;
        this.cellsCreated++;
      } else {
        this.cellsAlive--;
        this.cellsDeath++;
      }
    },
  },
};
</script>

<style>
.game-grid {
  background:#000c17;
  border-top: 1px solid #001529;
  border-left: 1px solid #001529;
  display: flex;
  flex: 1;
  justify-content: center;
  margin: 0 auto;
}
.game-column {
  flex: 1;
  display: flex;
  justify-content: center;
  padding: 0;
  margin: 0 auto;
  flex-direction: column;
}
</style>