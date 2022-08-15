<template>
  <div>

    <table>
      <tr v-for="(row, serialIndex) in rows" :key="serialIndex">
        <th>{{ row.title }}</th>
        <td v-for="(pos, posIndex) in row.cells" :key="posIndex">
          <div :class="{'node-holder': pos.isNode}" :style="cssColor()" v-if="pos.isNode">
            <div class="left-top-corner hidden"></div>
            <div class="top" :class="{hidden: pos.IncomingTop  !== true}"><img src="../assets/arrow.svg" alt="arrow"
                                                                               class="arrow"></div>
            <div class="right-top-corner hidden"></div>
            <div class="left" :class="{hidden: pos.OutgoingLeft !==true}"><img src="../assets/line.svg" alt="arrow"
                                                                               class="line"></div>
            <div class="node"></div>
            <div class="right" :class="{hidden: pos.OutgoingRight !== true}"><img src="../assets/line.svg" alt="arrow"
                                                                                  class="line"></div>
            <div class="left-bottom-corner"></div>
            <div class="bottom" :class="{hidden: pos.IncomingBottom  !== true}"><img src="../assets/arrow.svg"
                                                                                     alt="arrow" class="arrow"></div>
            <div class="right-bottom-corner"></div>
          </div>

          <div v-else class="arc-holder" :class="{'corner': pos.isForwardCorner && pos.isCorner,
          'revert-corner': !pos.isForwardCorner && pos.isCorner,
          'horizontal-line': pos.isHorizontalLine && !pos.isCorner,
          'vertical-line': pos.isVerticalLine  && !pos.isCorner}">
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
  for (let i = min; i <= max; i += step) colors.push('hsl(' + i + ',70%,50%)');
  return [...colors.sort(() => Math.random() - 0.5), ...colors.sort(() => Math.random() - 0.7)]
};

const colorsList = colorGenerator();
const cssColor = () => {
  return {'--bg-color': colorsList.pop()}
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
        color: colorsList.pop()
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

const connectNodes = (x, y, isForwardCorner) => {
  if (isForwardCorner) {
    for (let i = y + 1; i < x; i++) {
      rows[y].cells[i].isHorizontalLine = true;
      rows[i].cells[x].isVerticalLine = true;
    }

    rows[x].cells[x].IncomingTop = true;
    rows[y].cells[y].OutgoingRight = true;
  } else {
    for (let i = x + 1; i < y; i++) {
      rows[i].cells[x].isVerticalLine = true;
      rows[y].cells[i].isHorizontalLine = true;
    }
    rows[x].cells[x].IncomingBottom = true;
    rows[y].cells[y].OutgoingLeft = true;
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

fillMap();
fillLines();
</script>

<style scoped>

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
  padding: 0.5rem;
  background: #F2F2F2;
  color: rgba(29, 29, 29, 0.99);
  transition: ease-in-out 0.3s;
}

tr:hover th {
  background: rgba(0, 0, 0, 0.1);
  transition: ease-in-out 0.3s;
}

tr:hover {
  background: rgba(0, 0, 0, 0.1);
  transition: ease-in-out 0.3s;
}

td {
  height: 5rem;
  width: 5rem;
}

tr {
  transition: ease-in-out 0.3s;
}


.row-positions {
  flex: 1
}

.node-holder {
  display: grid;
  width: 100%;
  height: 100%;
  grid-template-columns: repeat(4, 25%);
  grid-template-rows: repeat(4, 25%);
  grid-template-areas: "right-top-corner top top left-top-corner"
  "left node node right"
  "left node node right"
  "left-bottom-corner bottom bottom right-bottom-corner";
  box-sizing: border-box;
}

.node {
  grid-area: node;
  height: 100%;
  width: 100%;
  border: 5px solid var(--bg-color);
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
}

.right-bottom-corner {
  grid-area: right-bottom-corner;
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

</style>
