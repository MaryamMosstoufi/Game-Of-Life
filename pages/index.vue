<template>
  <a-layout class='app'>
    <a-layout-content :style="{ margin: '24px'}">
    
      <Grid
        v-if="mainComponent == 'gamePage'"
        :message="message"
        :import-token="importToken"
        :current-speed="speed"
        @exportToken="exportSession($event)" />
              
      <Controls
        :is-running="isRunning"
        @send="delegate($event)"/>  
      
      <!-- Import -->
      <a-modal 
        v-if="isImport" 
        v-model="isImport" 
        :class="isImport ? 'is-active' : 'inactive'"
        title="Import"
        >
        <template slot="footer">
          <a-button key="back" @click="isImport = false">
            Cancel
          </a-button>
          <a-button key="submit" type="primary" :loading="loading" @click="importSession">
            Import
          </a-button>
        </template>
        <a-textarea
          v-model="importToken"
          type="text"
          style="margin-bottom: 24px"
          placeholder="Active Cells Array" />
        <select v-model="selectedScenario">
          <option value="scenario" selected>Pattern</option>
          <option value="achimpsp11">Achimpsp11</option>
          <option value="aforall">Aforall</option>
          <option value="alternatepentadecathlononsnacker">Alternate P on Snacker</option>
          <option value="koksgalaxy">Koksgalaxy</option>
          <option value="originalgliderbythedozen">Original Glider</option>
          <option value="gosper">Gosper glider gun</option>
          <option value="multiple">Multiple patterns</option>
          <option value="chess">Chess board</option>
          <option value="mandala">Mandala</option>
        </select>
        <!-- <button
          class="delete"
          aria-label="close"
          @click="isImport = false" /> -->
      </a-modal>   
      <!-- Export -->       
      <a-modal 
        v-if="isExport" 
        v-model="isExport" 
        title="Export"
        >
        <template slot="footer">
          <a-button key="back" @click="isExport = false">
            Cancel
          </a-button>
          <a-button key="submit" type="primary" :loading="loading" @click="toClipboard">
            Export
          </a-button>
        </template>
        <a-textarea
          id="copystring"
          v-model="exportToken"
          type="text"
          style="margin-bottom: 24px"
          placeholder="Fill Some Cells To Export"
          @focus="$event.target.select()" />
        <small>* By choosing export the array will be copied to clipboard.</small>
      </a-modal>
    </a-layout-content>
    <Footer/>
  </a-layout>
 
</template>

<script>
import Vue from "vue";
import Grid from "~/components/Grid.vue";
import Controls from "~/components/Controls.vue";
import Footer from "~/components/Footer.vue";
import { setInterval, clearInterval } from "timers";
export default {
  name: "App",
  components: {
    Grid,
    Controls,
  },
  props: {
    currentGeneration: {
      default: 0,
      type: Number,
    },
    cellCount: {
      default: 0,
      type: Number,
    },
    cellsAlive: {
      default: 0,
      type: Number,
    },
    cellsCreated: {
      default: 0,
      type: Number,
    },
    currentSpeed: {
      default: 0,
      type: Number,
    },
  },
  data() {
    return {
      message: "",
      importToken: "",
      exportToken: "",
      isRunning: false,
      isImport: false,
      isExport: false,
      speed: 100,
      intervalID: 0,
      mainComponent: "gamePage",
      selectedScenario: "scenario",
    };
  },
  watch: {
    selectedScenario: function (scenario) {
      if (scenario === "scenario") {
        this.importToken = "";
      } else if (scenario === "gosper") {
        this.importToken =
          "[5,13][5,14][6,13][6,14][15,10][15,11][15,15][15,16][16,11][16,12][16,13][16,14][16,15][17,11][17,12][17,14][17,15][18,11][18,12][18,14][18,15][19,12][19,13][19,14][27,15][28,14][28,15][28,16][29,13][29,14][29,15][29,16][29,17][30,12][30,14][30,16][30,18][31,12][31,13][31,17][31,18][39,14][39,15][40,14][40,15]";
      } else if (scenario === "multiple") {
        this.importToken =
          "[1,13][2,2][2,3][2,7][2,8][2,14][3,2][3,3][3,7][3,14][4,7][4,14][7,2][8,1][8,3][8,6][8,7][8,8][8,9][9,2][12,2][13,1][13,3][13,8][14,1][14,3][14,8][14,9][15,2][15,7][18,2][18,3][19,1][19,4][20,1][20,3][21,2][26,4][27,3][27,4][28,4][33,4][34,4][34,5][35,4][35,5][36,5][39,4][40,4][41,4]";
      } else if (scenario == "chess") {
        this.importToken =
          "[0,0][0,2][0,4][0,6][0,8][0,10][0,12][0,14][0,16][0,18][1,1][1,3][1,5][1,7][1,9][1,11][1,13][1,15][1,17][1,19][2,0][2,2][2,4][2,6][2,8][2,10][2,12][2,14][2,16][2,18][3,1][3,3][3,5][3,7][3,9][3,11][3,13][3,15][3,17][3,19][4,0][4,2][4,4][4,6][4,8][4,10][4,12][4,14][4,16][4,18][5,1][5,3][5,5][5,7][5,9][5,11][5,13][5,15][5,17][5,19][6,0][6,2][6,4][6,6][6,8][6,10][6,12][6,14][6,16][6,18][7,1][7,3][7,5][7,7][7,9][7,11][7,13][7,15][7,17][7,19][8,0][8,2][8,4][8,6][8,8][8,10][8,12][8,14][8,16][8,18][9,1][9,3][9,5][9,7][9,9][9,11][9,13][9,15][9,17][9,19][10,0][10,2][10,4][10,6][10,8][10,10][10,12][10,14][10,16][10,18][11,1][11,3][11,5][11,7][11,9][11,11][11,13][11,15][11,17][11,19][12,0][12,2][12,4][12,6][12,8][12,10][12,12][12,14][12,16][12,18][13,1][13,3][13,5][13,7][13,9][13,11][13,13][13,15][13,17][13,19][14,0][14,2][14,4][14,6][14,8][14,10][14,12][14,14][14,16][14,18][15,1][15,3][15,5][15,7][15,9][15,11][15,13][15,15][15,17][15,19][16,0][16,2][16,4][16,6][16,8][16,10][16,12][16,14][16,16][16,18][17,1][17,3][17,5][17,7][17,9][17,11][17,13][17,15][17,17][17,19][18,0][18,2][18,4][18,6][18,8][18,10][18,12][18,14][18,16][18,18][19,1][19,3][19,5][19,7][19,9][19,11][19,13][19,15][19,17][19,19][20,0][20,2][20,4][20,6][20,8][20,10][20,12][20,14][20,16][20,18][21,1][21,3][21,5][21,7][21,9][21,11][21,13][21,15][21,17][21,19][22,0][22,2][22,4][22,6][22,8][22,10][22,12][22,14][22,16][22,18][23,1][23,3][23,5][23,7][23,9][23,11][23,13][23,15][23,17][23,19][24,0][24,2][24,4][24,6][24,8][24,10][24,12][24,14][24,16][24,18][25,1][25,3][25,5][25,7][25,9][25,11][25,13][25,15][25,17][25,19][26,0][26,2][26,4][26,6][26,8][26,10][26,12][26,14][26,16][26,18][27,1][27,3][27,5][27,7][27,9][27,11][27,13][27,15][27,17][27,19][28,0][28,2][28,4][28,6][28,8][28,10][28,12][28,14][28,16][28,18][29,1][29,3][29,5][29,7][29,9][29,11][29,13][29,15][29,17][29,19][30,0][30,2][30,4][30,6][30,8][30,10][30,12][30,14][30,16][30,18][31,1][31,3][31,5][31,7][31,9][31,11][31,13][31,15][31,17][31,19][32,0][32,2][32,4][32,6][32,8][32,10][32,12][32,14][32,16][32,18][33,1][33,3][33,5][33,7][33,9][33,11][33,13][33,15][33,17][33,19][34,0][34,2][34,4][34,6][34,8][34,10][34,12][34,14][34,16][34,18][35,1][35,3][35,5][35,7][35,9][35,11][35,13][35,15][35,17][35,19][36,0][36,2][36,4][36,6][36,8][36,10][36,12][36,14][36,16][36,18][37,1][37,3][37,5][37,7][37,9][37,11][37,13][37,15][37,17][37,19][38,0][38,2][38,4][38,6][38,8][38,10][38,12][38,14][38,16][38,18][39,1][39,3][39,5][39,7][39,9][39,11][39,13][39,15][39,17][39,19][40,0][40,2][40,4][40,6][40,8][40,10][40,12][40,14][40,16][40,18][41,1][41,3][41,5][41,7][41,9][41,11][41,13][41,15][41,17][41,19][42,0][42,2][42,4][42,6][42,8][42,10][42,12][42,14][42,16][42,18][43,1][43,3][43,5][43,7][43,9][43,11][43,13][43,15][43,17][43,19][44,0][44,2][44,4][44,6][44,8][44,10][44,12][44,14][44,16][44,18][45,1][45,3][45,5][45,7][45,9][45,11][45,13][45,15][45,17][45,19][46,0][46,2][46,4][46,6][46,8][46,10][46,12][46,14][46,16][46,18][47,1][47,3][47,5][47,7][47,9][47,11][47,13][47,15][47,17][47,19][48,0][48,2][48,4][48,6][48,8][48,10][48,12][48,14][48,16][48,18][49,1][49,3][49,5][49,7][49,9][49,11][49,13][49,15][49,17][49,19][50,0][50,2][50,4][50,6][50,8][50,10][50,12][50,14][50,16][50,18][51,1][51,3][51,5][51,7][51,9][51,11][51,13][51,15][51,17][51,19][52,0][52,2][52,4][52,6][52,8][52,10][52,12][52,14][52,16][52,18][53,1][53,3][53,5][53,7][53,9][53,11][53,13][53,15][53,17][53,19][54,0][54,2][54,4][54,6][54,8][54,10][54,12][54,14][54,16][54,18][55,1][55,3][55,5][55,7][55,9][55,11][55,13][55,15][55,17][55,19][56,0][56,2][56,4][56,6][56,8][56,10][56,12][56,14][56,16][56,18][57,1][57,3][57,5][57,7][57,9][57,11][57,13][57,15][57,17][57,19][58,0][58,2][58,4][58,6][58,8][58,10][58,12][58,14][58,16][58,18][59,1][59,3][59,5][59,7][59,9][59,11][59,13][59,15][59,17][59,19][60,0][60,2][60,4][60,6][60,8][60,10][60,12][60,14][60,16][60,18][61,1][61,3][61,5][61,7][61,9][61,11][61,13][61,15][61,17][61,19][62,0][62,2][62,4][62,6][62,8][62,10][62,12][62,14][62,16][62,18][63,1][63,3][63,5][63,7][63,9][63,11][63,13][63,15][63,17][63,19][64,0][64,2][64,4][64,6][64,8][64,10][64,12][64,14][64,16][64,18][65,1][65,3][65,5][65,7][65,9][65,11][65,13][65,15][65,17][65,19][66,0][66,2][66,4][66,6][66,8][66,10][66,12][66,14][66,16][66,18][67,1][67,3][67,5][67,7][67,9][67,11][67,13][67,15][67,17][67,19][68,0][68,2][68,4][68,6][68,8][68,10][68,12][68,14][68,16][68,18][69,1][69,3][69,5][69,7][69,9][69,11][69,13][69,15][69,17][69,19][70,0][70,2][70,4][70,6][70,8][70,10][70,12][70,14][70,16][70,18][71,1][71,3][71,5][71,7][71,9][71,11][71,13][71,15][71,17][71,19][72,0][72,2][72,4][72,6][72,8][72,10][72,12][72,14][72,16][72,18][73,1][73,3][73,5][73,7][73,9][73,11][73,13][73,15][73,17][73,19][74,0][74,2][74,4][74,6][74,8][74,10][74,12][74,14][74,16][74,18][75,1][75,3][75,5][75,7][75,9][75,11][75,13][75,15][75,17][75,19][76,0][76,2][76,4][76,6][76,8][76,10][76,12][76,14][76,16][76,18][77,1][77,3][77,5][77,7][77,9][77,11][77,13][77,15][77,17][77,19][78,0][78,2][78,4][78,6][78,8][78,10][78,12][78,14][78,16][78,18][79,1][79,3][79,5][79,7][79,9][79,11][79,13][79,15][79,17][79,19][80,0][80,2][80,4][80,6][80,8][80,10][80,12][80,14][80,16][80,18][81,1][81,3][81,5][81,7][81,9][81,11][81,13][81,15][81,17][81,19][82,0][82,2][82,4][82,6][82,8][82,10][82,12][82,14][82,16][82,18][83,1][83,3][83,5][83,7][83,9][83,11][83,13][83,15][83,17][83,19][84,0][84,2][84,4][84,6][84,8][84,10][84,12][84,14][84,16][84,18][85,1][85,3][85,5][85,7][85,9][85,11][85,13][85,15][85,17][85,19][86,0][86,2][86,4][86,6][86,8][86,10][86,12][86,14][86,16][86,18][87,1][87,3][87,5][87,7][87,9][87,11][87,13][87,15][87,17][87,19][88,0][88,2][88,4][88,6][88,8][88,10][88,12][88,14][88,16][88,18][89,1][89,3][89,5][89,7][89,9][89,11][89,13][89,15][89,17][89,19][90,0][90,2][90,4][90,6][90,8][90,10][90,12][90,14][90,16][90,18][91,1][91,3][91,5][91,7][91,9][91,11][91,13][91,15][91,17][91,19][92,0][92,2][92,4][92,6][92,8][92,10][92,12][92,14][92,16][92,18][93,1][93,3][93,5][93,7][93,9][93,11][93,13][93,15][93,17][93,19][94,0][94,2][94,4][94,6][94,8][94,10][94,12][94,14][94,16][94,18][95,1][95,3][95,5][95,7][95,9][95,11][95,13][95,15][95,17][95,19][96,0][96,2][96,4][96,6][96,8][96,10][96,12][96,14][96,16][96,18][97,1][97,3][97,5][97,7][97,9][97,11][97,13][97,15][97,17][97,19][98,0][98,2][98,4][98,6][98,8][98,10][98,12][98,14][98,16][98,18][99,1][99,3][99,5][99,7][99,9][99,11][99,13][99,15][99,17][99,19]";
      } else if (scenario == "mandala") {
        this.importToken =
          "[13,9][13,10][14,2][14,3][14,4][14,8][14,11][14,15][14,16][14,17][15,9][15,10][16,6][16,13][17,6][17,7][17,12][17,13][18,6][18,13][20,2][20,3][20,4][20,15][20,16][20,17][25,2][25,3][25,4][25,15][25,16][25,17][27,6][27,13][28,6][28,7][28,12][28,13][29,6][29,13][30,9][30,10][31,2][31,3][31,4][31,8][31,11][31,15][31,16][31,17][32,9][32,10]";
      } else if (scenario == "achimpsp11") {
        this.importToken = 
          "[39,19][39,20][40,19][40,20][41,14][41,25][42,13][42,15][42,24][42,26][43,12][43,14][43,18][43,21][43,25][43,27][44,11][44,13][44,18][44,21][44,26][44,28][45,12][45,18][45,21][45,27][48,13][48,14][48,15][48,24][48,25][48,26][49,9][49,10][49,29][49,30][50,9][50,10][50,29][50,30][51,13][51,14][51,15][51,24][51,25][51,26][54,12][54,18][54,21][54,27][55,11][55,13][55,18][55,21][55,26][55,28][56,12][56,14][56,18][56,21][56,25][56,27][57,13][57,15][57,24][57,26][58,14][58,25][59,19][59,20][60,19][60,20]";
      } else if (scenario == "aforall") {
        this.importToken = 
          "[45,19][45,20][46,18][46,21][48,16][48,17][48,18][48,21][48,22][48,23][49,15][49,17][49,22][49,24][50,15][50,17][50,22][50,24][51,16][51,17][51,18][51,21][51,22][51,23][53,18][53,21][54,19][54,20]";
      } else if (scenario =="alternatepentadecathlononsnacker"){
        this.importToken =
          "[40,17][40,27][41,17][41,18][41,19][41,25][41,26][41,27][42,20][42,24][43,19][43,20][43,24][43,25][45,14][45,22][46,14][46,22][47,13][47,15][47,21][47,23][48,14][48,22][49,14][49,22][50,14][50,22][51,14][51,22][52,13][52,15][52,21][52,23][53,14][53,22][54,14][54,22][56,19][56,20][56,24][56,25][57,20][57,24][58,17][58,18][58,19][58,25][58,26][58,27][59,17][59,27]";
      } else if (scenario =="koksgalaxy") {
        this.importToken =
          "[45,17][45,19][45,22][46,17][46,18][46,19][46,21][46,23][46,24][47,16][47,23][48,17][48,23][48,24][50,16][50,17][50,23][51,17][51,24][52,16][52,17][52,19][52,21][52,22][52,23][53,18][53,21][53,23]";
      } else if (scenario =="originalgliderbythedozen"){
        this.importToken =
          "[46,20][47,19][47,21][48,18][48,22][49,18][49,19][49,20][49,21][49,22][51,18][51,19][51,20][51,21][51,22][52,18][52,22][53,19][53,21][54,20]";
      }
    },
  },
  created() {},
  methods: {
    delegate: function (event) {
      if (event === "play") {
        this.isRunning = !this.isRunning;
        this.restartInterval();
      } else if (event === "importSession") {
        this.isImport = true;
        this.visible = true;
      } else if (event === "exportSession") {
        this.updateMessage("exportSession");
      } else if (event === "slowDown") {
        this.speed > 100 ? this.changeSpeed(-100) : this.changeSpeed(-20);
        this.restartInterval();
      } else if (event === "speedUp") {
        this.speed < 100 ? this.changeSpeed(20) : this.changeSpeed(100);
        this.restartInterval();
      } else {
        this.updateMessage(event);
      }
    },
    updateMessage: function (newMessage) {
      this.message = newMessage;
      Vue.nextTick(this.resetMessage);
    },
    resetMessage: function () {
      this.message = "";
    },
    restartInterval: function () {
      clearInterval(this.intervalID);
      if (this.isRunning) {
        this.intervalID = setInterval(
          this.updateMessage,
          50000 / this.speed,
          "nextStep"
        );
      }
    },
    changeSpeed: function (speed) {
      this.speed += speed;
      if (this.speed < 20) {
        this.speed = 20;
      } else if (this.speed > 500) {
        this.speed = 500;
      }
    },
    importSession: function () {
      this.updateMessage("importSession");
      this.isImport = false;
    },
    exportSession: function (exportToken) {
      this.exportToken = exportToken;
      this.isExport = true;
    },
    toClipboard: function () {
      this.isExport = false;
      let copyString = document.querySelector("#copystring");
      copyString.setAttribute("type", "text");
      copyString.select();
      document.execCommand("copy");
    },
  },
};
</script>

<style>
.app{
  min-height: 100vh;
}
.ant-layout,
.ant-layout-content {
  background:#001529;
}
</style>
