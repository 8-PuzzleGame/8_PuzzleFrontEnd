<template>
  <div class="puzzle-comp-cont">
    <div class="navImgs">
      <img id="view" src="@/assets/view.jpg" alt="view" @click="drawImg" />
      <img id="jumb" src="@/assets/jumb.jpg" alt="jumb" @click="drawImg" />
      <img id="anime" src="@/assets/anime.jpg" alt="anime" @click="drawImg" />
    </div>
    <div class="content">
      <div class="options">
        <div>
          <input class="show-nums" type="checkbox" v-model="show_numbers" />
          <label for="checkbox">show numbers</label>
        </div>
        <div>
          <label for="algorithm">Algorithm:</label>
          <select class="algorithm" v-model="algorithm">
            <option value="A_star">A* h1</option>
            <option value="A_starh2">A* h2</option>
            <option value="BFS">BFS</option>
            <option value="DFS">DFS</option>
          </select>
        </div>
      </div>
      <div class="game">
        <canvas id="backCnv" :width="cnvL" :height="cnvL"></canvas>
        <div class="puzzle">
          <div
            class="section"
            v-for="i in 9"
            :key="i"
            :id="'section-' + (i - 1)"
            @click="swapSectionClick(i - 1)"
          >
            <canvas class="imgPart" :width="partL" :height="partL"></canvas>
            <span v-show="show_numbers">{{ i }}</span>
          </div>
        </div>
      </div>
      <div class="btns">
        <input
          class="custom_state"
          v-model="custom_state"
          placeholder="Custom State"
        />
        <div class="button1" @click="shuffleThePuzzle(false)">Shuffle</div>
        <div
          class="button1 btn-sol"
          :style="'background-color:' + solBtncolor"
          @click="solveThePuzzle"
        >
          {{ solBtn }}
        </div>
      </div>
      <div class="sol-info">
        <div class="path" v-show="path" v-html="path"></div>
        <textarea class="algorithm-info" v-show="info" v-text="info"></textarea>
      </div>
      <div class="message" v-html="message"></div>
    </div>
  </div>
</template>

<script>
/*
example is
list <= [3, 0, 1, 8, 5, 2, 6, 4, 7]
puzzle <= [4, 1, 2, 0, 6, 3, 7, 5, 8]
path <= [Up, Right, Right, Down, Left, Down, Right]
run
\u25B6
stop
\u25FC
*/
import { gsap } from "gsap";

const cnvL = 600,
  partL = cnvL / 3,
  emp = 8;

let grid = [0, 1, 2, 3, 4, 5, 6, 7, 8],
  indexs = [0, 1, 2, 3, 4, 5, 6, 7, 8],
  animationFlag = false,
  runFlag = false,
  animation = {
    id: null,
    interval: 600,
    path: null,
    pathIndex: 0,
  },
  sections;

function swapSection(e) {
  const eIndex = indexs[e];
  let step = 0;

  if (grid[eIndex - 3] == emp) step = -3;
  else if (grid[eIndex + 1] == emp) step = 1;
  else if (grid[indexs[e] + 3] == emp) step = 3;
  else if (grid[indexs[e] - 1] == emp) step = -1;
  else return;

  indexs[e] += step;
  swap(grid, eIndex, eIndex + step);
  flip(sections[e], () => {
    moveSection(e, indexs[e]);
    moveSection(emp, eIndex);
  });
}
function swap(list, i1, i2) {
  const temp = list[i1];
  list[i1] = list[i2];
  list[i2] = temp;
}
function moveSection(index, position) {
  sections[index].style.setProperty("grid-area", `a${position}`);
}
function isSolvable(list) {
  let inversions = 0;
  for (let i = 0; i < list.length - 1; i++) {
    if (list[i] == 8) continue;
    for (let j = i + 1; j < list.length; j++)
      if (list[j] != 8 && list[i] > list[j]) inversions++;
  }
  return inversions % 2 == 0;
}
function shuffleList(list) {
  do {
    for (let i = list.length - 1; i > 0; i--) {
      swap(list, i, Math.floor(Math.random() * (i + 1)));
    }
  } while (!isSolvable(list));
  return list;
}
function orderSections(list) {
  grid = list;
  for (let i = 0; i < grid.length; i++) {
    indexs[grid[i]] = i;
    moveSection(grid[i], i);
  }
}
function checkWin() {
  return String(grid) == String([0, 1, 2, 3, 4, 5, 6, 7, 8]);
}

function flip(elements, changeFunc, vars) {
  elements = gsap.utils.toArray(elements);
  vars = vars || {};
  let tl = gsap.timeline({
      onComplete: vars.onComplete,
      delay: vars.delay || 0,
    }),
    bounds = elements.map((el) => el.getBoundingClientRect()),
    copy = {},
    p;
  elements.forEach((el) => {
    el._flip && el._flip.progress(1);
    el._flip = tl;
  });
  changeFunc();
  for (p in vars) {
    p !== "onComplete" && p !== "delay" && (copy[p] = vars[p]);
  }
  copy.x = (i, element) =>
    "+=" + (bounds[i].left - element.getBoundingClientRect().left);
  copy.y = (i, element) =>
    "+=" + (bounds[i].top - element.getBoundingClientRect().top);
  return tl.from(elements, copy);
}
export default {
  name: "8-puzzle",
  data: function () {
    return {
      cnvL: cnvL,
      partL: partL,
      message: "",
      show_numbers: false,
      algorithm: "A_star",
      solBtn: "Solution",
      solBtncolor: "red",
      path: "",
      info: "",
      custom_state: "",
    };
  },
  mounted() {
    sections = document.getElementsByClassName("section");
  },
  methods: {
    init_puzzle() {
      grid = [0, 1, 2, 3, 4, 5, 6, 7, 8];
      orderSections(grid);
      this.message = "";
      this.path = "";
      this.info = "";
      this.endAnimation();
    },
    drawImg(e) {
      // scale image
      const scaledImg = document.createElement("canvas");
      scaledImg.width = cnvL;
      scaledImg.height = cnvL;
      scaledImg.getContext("2d").drawImage(e.srcElement, 0, 0, cnvL, cnvL);
      // set background
      document
        .getElementById("backCnv")
        .getContext("2d")
        .drawImage(scaledImg, 0, 0, cnvL, cnvL);
      // split and draw
      const imgParts = document.getElementsByClassName("imgPart"),
        imgPartL = cnvL / 3;
      for (let i = 0; i < 9; i++) {
        imgParts[i]
          .getContext("2d")
          .drawImage(
            scaledImg,
            imgPartL * (i % 3),
            imgPartL * Math.floor(i / 3),
            imgPartL,
            imgPartL,
            0,
            0,
            partL,
            partL
          );
      }
      this.init_puzzle();
    },
    swapSectionClick(e) {
      if (animationFlag) return;

      swapSection(e);
      if (checkWin())
        this.message = `<span>Well Done <span style="font-size: 1.2em;">&#9745;</span></span>`;
    },
    shuffleThePuzzle(custom, state) {
      if (runFlag) return;

      this.init_puzzle();
      if (custom) grid = state;
      else grid = shuffleList(grid);

      orderSections(grid);
      this.message = "";
    },
    solveThePuzzle() {
      if (!animationFlag) {
        if (checkWin()) {
          console.log("Already Solved");
          return;
        }
        let state = "";
        for (let num of grid) state = state + String((num + 1) % 9);

        let method_id = 1;
        if (this.algorithm === "DFS") method_id = 2;
        else if (this.algorithm === "A_star") method_id = 3;
        else if (this.algorithm === "A_starh2") method_id = 4;
        this.puzzleSolution(state, "123456780", method_id);
      } else if (runFlag) {
        this.stopAnimation();
        this.solBtnStyle("Run", "green");
        runFlag = false;
      } else {
        this.runAnimation();
        this.solBtnStyle("Pause", "red");
        runFlag = true;
      }
    },
    solBtnStyle(name, color) {
      this.solBtn = name;
      this.solBtncolor = color;
    },
    runAnimation() {
      let empIndex = grid.indexOf(8);

      animation.id = setInterval(() => {
        let move = animation.path[animation.pathIndex];
        let step = 0;

        if (move === "Up") step = -3;
        else if (move === "Down") step = 3;
        else if (move === "Right") step = 1;
        else if (move === "Left") step = -1;
        else {
          this.endAnimation();
          return;
        }

        swapSection(grid[empIndex + step]);
        empIndex += step;
        animation.pathIndex++;

        if (checkWin()) this.endAnimation();
      }, animation.interval);
    },
    stopAnimation() {
      clearInterval(animation.id);
    },
    endAnimation() {
      this.stopAnimation();
      this.solBtnStyle("Solution", "red");
      animation.path = [];
      runFlag = false;
      animationFlag = false;
    },
    puzzleSolution(state, goal, method_id) {
      let sol = null;
      let a = {
        state: state,
        goal: goal,
        method_id: method_id,
      };
      fetch("http://localhost:8080//Solve", {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
        },
        body: JSON.stringify(a),
      })
        .then((response) => response.text())
        .then((data) => {
          let solObj = JSON.parse(data);
          console.log(solObj);

          this.path = solObj.path.length == 0 ? "No Solution!" : solObj.path;

          this.info =
            `Cost: ${solObj.cost}\n` +
            `Search Depth: ${solObj.search_depth}\n` +
            `Nodes Expanded: ${solObj.nodes_expanded}\n` +
            `Time: ${solObj.time} ms`;

          animation.path = this.path;
          animation.pathIndex = 0;

          this.solBtnStyle("Run", "green");
          animationFlag = true;
        });
      return sol;
    },
  },
  watch: {
    custom_state: function () {
      const str = this.custom_state;
      const nums = [0, 1, 2, 3, 4, 5, 6, 7, 8];

      let checkLen = str.length == 9;
      let digitsOnly = /^[0-8]+$/.test(str);

      if (!(checkLen && digitsOnly)) return;

      for (let ch of str) nums[Number(ch)] = -1;

      let sum = 0;
      for (let n of nums) sum += Number(n);

      let state = [0, 0, 0, 0, 0, 0, 0, 0, 0];
      if (sum == -9) {
        for (let i = 0; i < str.length; i++)
          state[i] = Number(str[i]) === 0 ? 8 : Number(str[i]) - 1;
        this.shuffleThePuzzle(true, state);
      }
    },
  },
};
</script>

<style scoped>
/* layout style */
.puzzle-comp-cont {
  display: flex;
  height: 100vh;
}
.navImgs {
  display: flex;
  padding: 20px;
  width: 25%;
  max-width: 250px;
  gap: 10px;
  flex-direction: column;
  overflow: auto;
  background-color: var(--colorD2);
}
.navImgs img {
  border-radius: 1em;
}
.content {
  padding: 30px 10px 10px 10px;
  flex: 1;
  overflow: auto;
}
.game {
  position: relative;
  margin: auto;
  width: 400px;
  height: 400px;
}
.btns div {
  margin: 10px 10px 10px 0;
  user-select: none;
}
.custom_state {
  display: block;
  margin: 10px auto 0 auto;
  padding: 5px;
}
.sol-info {
  margin: auto;
  width: 90%;
}
.options {
  display: inline-flex;
  width: 400px;
  flex-direction: row;
  justify-content: space-between;
}
.show-nums {
  margin: 0 5px 5px 0;
}
.algorithm {
  margin: 0 0px 5px 5px;
}
.algorithm-info {
  padding: 5px;
  width: 100%;
  height: 100px;
  font-size: 1.2em;
  font-family: Arial, Helvetica, sans-serif;
  resize: none;
  color: var(--colorL1);
  background-color: var(--colorD2);
}
@media (max-width: 600px) {
  .puzzle-comp-cont {
    flex-direction: column;
    height: auto;
  }
  .navImgs {
    padding: 10px;
    flex-direction: row;
    width: 100%;
    max-width: none;
    height: 20vh;
  }
  .game {
    width: 90vw;
    height: 90vw;
  }
  .options {
    width: 90vw;
  }
}
/* game style*/
#backCnv {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  opacity: 0.3;
  background-color: var(--colorD2);
}
.puzzle {
  display: grid;
  grid-template-areas: "a0 a1 a2" "a3 a4 a5" "a6 a7 a8";
  gap: 1px 1px;
  width: 100%;
  height: 100%;
}
.section {
  display: flex;
  position: relative;
  align-items: center;
  justify-content: center;
}
.section * {
  position: absolute;
}
.section span {
  z-index: 1;
  font-size: 2em;
  user-select: none;
}
.section canvas {
  width: 100%;
  height: 100%;
}
#section-8 {
  visibility: hidden;
}
.message:deep() span {
  color: rgb(30, 255, 40);
}
.path {
  margin-bottom: 10px;
  padding: 5px;
  overflow-x: auto;
  white-space: nowrap;
  background-color: var(--colorD2);
}
</style>
