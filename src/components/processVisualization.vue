<template>
  <div>

    <table :style="cssColor()">
      <tr v-for="(row, serialIndex) in rows" :key="serialIndex">
        <th>{{ row.title }}</th>
        <td v-for="(pos, posIndex) in row.cells" :key="posIndex">
          <div :class="{'node-holder': pos.isNode}" v-if="pos.isNode">
            <div class="left-top-corner">
              <img
                  src="../assets/arrow-2.svg" alt="arrow"
                  class="arrow"
                  :class="{hidden: pos.isDiagonalLeftTopIn !== true, 'diagonal-top-incoming-arrow': pos.isDiagonalLeftTopIn === true}">
              <img src="../assets/line.svg" alt="line" :class="{hidden: pos.isDiagonalLeftTopOut !== true ||pos.isDiagonalLeftTopIn === true,
              'diagonal-line': pos.isDiagonalLeftTopOut } "
                   class="line-out">
            </div>
            <div class="top">
              <img src="../assets/arrow.svg" alt="arrow" class="arrow bottom-node"
                   :class="{hidden: pos.isInTop  !== true}">
              <img src="../assets/line.svg" alt="arrow" class="vertical-line-node"
                   :class="{hidden: pos.isOutTop !== true}">
            </div>
            <div class="right-top-corner hidden"></div>
            <div class="left" :class="{hidden: pos.isOutLeft !==true}"><img src="../assets/line.svg" alt="arrow"
                                                                            class="line"></div>
            <div class="node"></div>
            <div class="right" :class="{hidden: pos.isOutRight !== true}"><img src="../assets/line.svg" alt="arrow"
                                                                               class="line"></div>
            <div class="left-bottom-corner"></div>
            <div class="bottom">
              <img src="../assets/arrow.svg" alt="arrow" class="arrow bottom-node"
                   :class="{hidden: pos.isInBottom !== true}">
              <img src="../assets/line.svg" alt="arrow" class="vertical-line-node"
                   :class="{hidden: pos.isOutBottom !== true}">
            </div>
            <div class="right-bottom-corner">
              <img
                  src="../assets/arrow-2.svg" alt="arrow"
                  class="arrow"
                  :class="{hidden: pos.isDiagonalRightBottomIn !== true, 'incoming-right-bottom-corner': pos.isDiagonalRightBottomIn}">
              <img
                  src="../assets/line.svg" alt="arrow"
                  class="line diagonal-line"
                  :class="{hidden: pos.isDiagonalRightBottomOut !== true || pos.isDiagonalRightBottomIn === true}">
            </div>
          </div>

          <div v-else class="arc-holder" :class="{'corner': pos.isForwardCorner && pos.isCorner,
          'revert-corner': !pos.isForwardCorner && pos.isCorner,
          'horizontal-line': pos.isHorizontalLine ,
          'vertical-line': pos.isVerticalLine }">
            <div class="left-top"></div>
            <div class="right-top"></div>
            <div class="left-bottom"></div>
            <div class="right-bottom"></div>
          </div>
        </td>
      </tr>
    </table>
  </div>
</template>
<script>
export default {
  name: 'processVisualization',
}
</script>
<script setup>
import {defineProps} from "vue";

const props = defineProps({
  data: Object
});

const colorGenerator = () => {
  const max = 330;
  const min = 0;
  const step = 30;
  const colors = [];
  for (let i = min; i <= max; i += step) colors.push('hsl(' + i + ',65%,70%)');
  //33 цвета
  const colors33 = [...colors.sort(() => Math.random() - 0.5),
    ...colors.sort(() => Math.random() - 0.7),
    ...colors.sort(() => Math.random() - 0.3)]
  return [...colors33, ...colors33, ...colors33]
};
const getColor = () => {
  let newColor = colorsList.pop();
  if (newColor === undefined) {
    colorGenerator();
    newColor = colorsList.pop();
  }
  return newColor;
};
const colorsList = colorGenerator();
const cssColor = () => {
  return {'--random-color': colorsList.pop()}
}
const rows = [];
const fillMap = () => {
  for (let j = 0; j < props.data.states.length; j++) {
    const newRow = [];
    const rowAdjacencies = props.data.adjacencies[j];
    for (let i = 0; i < props.data.states.length; i++) {
      const isCorner = rowAdjacencies[i] === true;
      let isForwardCorner;
      const isNode = i === j;
      const pos = {
        x: i,
        y: j,
        isNode: isNode,
        isCorner: isCorner,
        color: getColor()
      };

      if (isCorner) {
        isForwardCorner = i > j
        pos.isForwardCorner = isForwardCorner;
      }
      newRow.push(pos);
    }
    rows.push({title: props.data.states[j], cells: newRow})
  }
};

const getNodeByRow = (rowIndex) => {
  return rows[rowIndex].cells.find(pos => pos.isNode === true);
};

const markedLineAsNodeTheNodeConnected = (x, row, node) => {
  //проверить есть ли массив
  //если этой ноды еще нет, добавить
  const node = "x" + node.x + "y" + node.y;
  if (rows[row].cells[x]?.connectedNodes) {
    if (!rows[row].cells[x].connectedNodes.includes(node)) {
      rows[row].cells[x].connectedNodes.push(node);
    }
  }
};
const connectNodes = (x, y, isForwardCorner) => {

  if (isForwardCorner) {
    for (let i = y + 1; i < x; i++) {
      rows[y].cells[i].isHorizontalLine = true;
      markedLineAsNodeTheNodeConnected(x, y, getNodeByRow(y));
      rows[i].cells[x].isVerticalLine = true;
      markedLineAsNodeTheNodeConnected(x, i, getNodeByRow(i));
    }
    rows[x].cells[x].isInTop = true;
    markedLineAsNodeTheNodeConnected(x, x, getNodeByRow(x));
    rows[y].cells[y].isOutRight = true;
    markedLineAsNodeTheNodeConnected(y, y, getNodeByRow(y));
  } else {
    for (let i = x + 1; i < y; i++) {
      rows[i].cells[x].isVerticalLine = true;
      markedLineAsNodeTheNodeConnected(x, i, getNodeByRow(i));
      rows[y].cells[i].isHorizontalLine = true;
      markedLineAsNodeTheNodeConnected(i, y, getNodeByRow(y));
    }
    rows[x].cells[x].isInBottom = true;
    markedLineAsNodeTheNodeConnected(x, x, getNodeByRow(x));
    rows[y].cells[y].isOutLeft = true;
    markedLineAsNodeTheNodeConnected(y, y, getNodeByRow(y));
  }
};

const makeDiagonals = () => {
  for (let i = 0; i < rows.length - 1; i++) {
    const node = rows[i].cells[i];
    const nextNode = rows[i + 1].cells[i + 1];
    if (rows[i].cells[i + 1].isForwardCorner === true) {
      if (!rows[i].cells[i + 1].isVerticalLine) {
        nextNode.isInTop = false;
      }
      if (!rows[i].cells[i + 1].isHorizontalLine) {
        node.isOutRight = false;
      }
      nextNode.isDiagonalLeftTopIn = true;
      node.isDiagonalRightBottomOut = true;
      rows[i].cells[i + 1].isForwardCorner = false;
      rows[i].cells[i + 1].isCorner = false;
    }
    if (!(rows[i + 1].cells[i].isForwardCorner === true) && rows[i + 1].cells[i].isCorner === true) {
      if (!rows[i + 1].cells[i].isVerticalLine) {
        node.isInBottom = false;
      }
      if (!rows[i + 1].cells[i].isHorizontalLine) {
        nextNode.isOutLeft = false;
      }
      node.isDiagonalRightBottomIn = true;
      nextNode.isDiagonalLeftTopOut = true;
      rows[i + 1].cells[i].isForwardCorner = false;
      rows[i + 1].cells[i].isCorner = false;
    }
  }
};

const fillLines = () => {
  rows.forEach(row => {
    row.cells.forEach(pos => {
      if (pos.isCorner) {
        connectNodes(pos.x, pos.y, pos.isForwardCorner)
      }
    })
  })
};

const processedNodesPoint = {
  x: -1,
  y: -1,
}

const protectedAreaPoint = {
  x: -2,
  y: rows.length - 1,
}

const getNodeIndexXByRow = (rowIndex) => {
  return rows[rowIndex].cells.find(pos => pos.isNode === true).x;
};

const isPossibleToMoveRight = (node) => {
  return node.x !== 0 && rows[node.y].cells[node.x - 1].isVerticalLine !== true && node.isInTop !== true;
};

const markedNodesAsProcessed = () => {
  processedNodesPoint.x += 1;
  processedNodesPoint.y += 1
};

const increaseProtectedArea = () => {
  protectedAreaPoint.x += 1;
};

const movePositionsToRight = (node) => {
  for (let i = 0; i < rows.length; i++) {
    for (let j = 0; j < rows[i].cells.length; j++) {
      const currentPos = {
        x: j,
        y: i
      }
      if ((currentPos.x > processedNodesPoint.x
              || currentPos.y > processedNodesPoint.y)
          && currentPos.x > protectedAreaPoint.x) {

        if (currentPos.x === rows[currentPos.y].cells.length - 1) {
          rows[currentPos.y].cells.pop();
        } else {
          rows[currentPos.y].cells[currentPos.x] = JSON.parse(JSON.stringify(rows[currentPos.y].cells[currentPos.x + 1]));
          rows[currentPos.y].cells[currentPos.x].x -= 1;
        }

        if (currentPos.x === node.x - 1 && currentPos.y === node.y) {
          if (rows[currentPos.y].cells[currentPos.x]?.isDiagonalLeftTopIn === true) {
            rows[currentPos.y].cells[currentPos.x].isDiagonalLeftTopIn = false;
            rows[currentPos.y].cells[currentPos.x].isInTop = true;

            const prevNodeXIndex = getNodeIndexXByRow(currentPos.y - 1);
            rows[currentPos.y - 1].cells[prevNodeXIndex].isDiagonalRightBottomOut = false;
            rows[currentPos.y - 1].cells[prevNodeXIndex].isOutBottom = true;
          }

          if (rows[currentPos.y].cells[currentPos.x]?.isDiagonalLeftTopOut === true) {
            rows[currentPos.y].cells[currentPos.x].isDiagonalLeftTopOut = false;
            rows[currentPos.y].cells[currentPos.x].isOutTop = true;

            const prevNodeXIndex = getNodeIndexXByRow(currentPos.y - 1);
            rows[currentPos.y - 1].cells[prevNodeXIndex].isDiagonalRightBottomIn = false;
            rows[currentPos.y - 1].cells[prevNodeXIndex].isInBottom = true;
          }
        }
      }
    }
  }
};

const processNodesForMoving = () => {
  for (let i = 0; i < rows.length; i++) {
    const currentNode = getNodeByRow(i);
    if (isPossibleToMoveRight(currentNode)) {
      movePositionsToRight(currentNode);
      processedNodesPoint.y += 1;
    } else {
      markedNodesAsProcessed();
      increaseProtectedArea();
    }
  }
};

fillMap();
fillLines();
makeDiagonals();
processNodesForMoving();
console.log(rows)
</script>

<style scoped>

* {
  margin: 0;
  padding: 0;
  font-family: "Arial Narrow", sans-serif;
  font-size: medium;
  font-weight: 200;
}

/*:root {*/
/*  --background-color: red;*/
/*}*/
/*
grey #e1e2e5
dark blue #4981fd

*/
table {
  box-shadow: 0 14px 28px rgba(0, 0, 0, 0.25), 0 10px 10px rgba(0, 0, 0, 0.22);
}

table, th {
  border: 0.5px solid lightgrey;
  border-collapse: collapse;
}

th {
  padding: 1rem;
  background: #F2F2F2;
  color: rgba(29, 29, 29, 0.99);
  transition: ease-in-out 0.4s;
}

tr:hover th {
  background: rgba(0, 0, 0, 0.3);
  transition: ease-in-out 0.4s;
}

tr:hover {
  background: rgba(0, 0, 0, 0.1);
  transition: ease-in-out 0.4s;
}

td {
  height: 5rem;
  width: 5rem;
}

tr {
  transition: ease-in-out 0.4s;
}

.node-holder {
  display: grid;
  width: 5rem;
  height: 5rem;
  grid-template-columns: repeat(4, 25%);
  grid-template-rows: repeat(4, 25%);
  grid-template-areas: "left-top-corner top top right-top-corner"
  "left node node right"
  "left node node right"
  "left-bottom-corner bottom bottom right-bottom-corner";
  box-sizing: border-box;
}

.node {
  grid-area: node;
  height: 100%;
  width: 100%;
  border: 5px solid red;
  border-radius: 50%;
  display: inline-block;
  box-sizing: border-box;
}

.right-top-corner {
  grid-area: right-top-corner;
}

.top, .bottom {
  display: flex;
  justify-content: center;
  align-items: center;
}

.top {
  grid-area: top;
  position: relative;
}

.arrow {
  height: 100%;
}

.top .arrow {
  transform: rotate(180deg);
}

.left-top-corner {
  grid-area: left-top-corner;
}

.left {
  grid-area: left;
}


.right, .left {
  display: flex;
  align-items: center;
  justify-content: center;
}

.right {
  grid-area: right;
}

.left-bottom-corner {
  grid-area: left-bottom-corner;
}

.bottom {
  grid-area: bottom;
  position: relative;
}

.right-bottom-corner {
  grid-area: right-bottom-corner;
}

.left-top-corner, .right-top-corner, .left-bottom-corner, .right-bottom-corner {
  position: relative;
}

.left-top-corner img, .right-top-corner img, .left-bottom-corner img, .right-bottom-corner img {
  position: absolute;
  top: 0;
  left: 0;
}

.arc-holder {
  display: grid;
  width: 100%;
  height: 100%;
  grid-template-columns: repeat(2, 50%);
  grid-template-rows: repeat(2, 50%);
  grid-template-areas: "left-top right-top" "left-bottom right-bottom";
  gap: 0;
}

.left-top {
  grid-area: left-top;
}

.right-top {
  grid-area: right-top;
}

.left-bottom {
  grid-area: left-bottom;
}

.right-bottom {
  grid-area: right-bottom;
}

.corner .left-top {
  border-bottom: 1px grey solid;
}

.corner .left-bottom {
  border-top: 1px grey solid;
  border-right: 1px grey solid;
}

.corner .right-bottom {
  border-left: 1px grey solid;
}


.revert-corner .left-top {
  border-right: 1px grey solid;
}

.revert-corner .right-top {
  border-bottom: 1px grey solid;
  border-left: 1px grey solid;
}

.revert-corner .right-bottom {
  border-top: 1px grey solid;
}


.horizontal-line .left-bottom, .horizontal-line .right-bottom {
  border-top: 1px grey solid;
}

.horizontal-line .right-top, .horizontal-line .left-top {
  border-bottom: 1px grey solid;
}

.vertical-line .right-bottom, .vertical-line .right-top {
  border-left: 1px grey solid;
}

.vertical-line .left-top, .vertical-line .left-bottom {
  border-right: 1px grey solid;
}

.hidden {
  visibility: hidden;
}

.incoming-right-bottom-corner {

}

.diagonal-top-incoming-arrow {
  width: 1rem;
  transform: rotate(180deg);
  z-index: 2;
}

.diagonal-bottom-incoming-arrow {
  width: 1rem;
  transform: rotate(180deg);
}

.diagonal-line {
  transform: rotate(45deg);
}

.bottom-node, .vertical-line-node {
  height: 1rem;
  position: absolute;
  top: 0;
  left: calc(50% - 0.5rem);
}

.vertical-line-node {
  transform: rotate(90deg);
}
</style>
