<template>
  <div>

    <table>
      <tr v-for="(row, serialIndex) in rows" :key="serialIndex">
        <th>{{ row.title }}</th>
        <td v-for="(pos, posIndex) in row.cells" :key="posIndex">
          <div :style="cssColor()" v-if="pos.isNode">
            <div :class="{node: pos.isNode}">
              <div class="right-top-corner"></div>
              <div class="top"><img src="../assets/arrow.svg" alt="arrow" class="arrow" :class="{hidden: pos.IncomingTop}"></div>
              <div class="left-top-corner"></div>
              <div class="left"></div>
              <div class="node"></div>
              <div class="right"></div>
              <div class="left-bottom-corner"></div>
              <div class="bottom"><img src="../assets/arrow.svg" alt="arrow" class="arrow arrow-bottom" :class="{hidden: pos.IncomingTop}"></div>
              <div class="right-bottom-corner"></div>
            </div>
          </div>

          <div v-else class="arc-holder">
            <div class="left-top"></div>
            <div class="right-top"></div>
            <div class="left-bottom"></div>
            <div class="right-bottom"></div>
          </div>


<!--          <div :style="cssColor()" v-if="pos.isNode" class="node-placeholder">-->
<!--            <div class="top">-->
<!--              <img src="../assets/arrow.svg" alt="arrow" class="arrow" :class="{hidden: pos.IncomingTop}">-->
<!--            </div>-->
<!--            <div class="middle">-->
<!--              <div class="outgoing-left" >-->
<!--               <div class='horizontal-line' :class="{hidden: pos.OutgoingLeft}">-->
<!--                 <div v-for="item in [1,2]" :key="item"></div>-->
<!--               </div>-->
<!--              </div>-->
<!--              <div :class="{node: pos.isNode}"></div>-->
<!--              <div class='horizontal-line' :class="{hidden: pos.OutgoingRight}">-->
<!--                <div v-for="item in [1,2]" :key="item"></div>-->
<!--              </div>-->
<!--            </div>-->
<!--            <div class="bottom">-->
<!--              <img src="../assets/arrow.svg" alt="arrow" class="arrow" :class="{hidden: pos.IncomingBottom}">-->
<!--            </div>-->
<!--          </div>-->
<!--          <div :class="{'corner': pos.isForwardCorner}" v-if="pos.isCorner && pos.isForwardCorner"></div>-->
<!--          <div :class="{'revert-corner': !pos.isForwardCorner}" v-if="pos.isCorner && !pos.isForwardCorner"></div>-->
<!--          <div v-if="pos.isHorizontalLine && !pos.isVerticalLine" :class="{'horizontal-line': pos.isHorizontalLine}"-->
<!--               class="line">-->
<!--            <div v-for="item in [1,2]" :key="item"></div>-->
<!--          </div>-->
<!--          <div v-if="pos.isVerticalLine && !pos.isHorizontalLine" :class="{'vertical-line': pos.isVerticalLine}"-->
<!--               class="line">-->
<!--            <div v-for="item in [1,2]" :key="item"></div>-->
<!--          </div>-->
<!--          <div :class="{'cross-line': pos.isVerticalLine && pos.isHorizontalLine}"-->
<!--               v-if="pos.isVerticalLine && pos.isHorizontalLine">-->
<!--            <div v-for="item in [1,2,3,4]" :key="item"></div>-->
<!--          </div>-->
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
  return [...colors.sort(() => Math.random() - 0.5), ...colors.sort(() => Math.random() - 0.5)]
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

    rows[y].cells[y].OutgoingRight = true;
    rows[x].cells[x].IncomingTop = true;
  } else {
    for (let i = x + 1; i < y; i++) {
      rows[i].cells[x].isVerticalLine = true;
      rows[y].cells[i].isHorizontalLine = true;
    }
    rows[y].cells[y].OutgoingLeft = true;
    rows[x].cells[x].IncomingBottom = true;
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
table, th {
  border: 0.5px solid lightgrey;
  border-collapse: collapse;

}

th {
  padding: 0.2rem;
  background: #F2F2F2;
  color: grey;
}

td {
  height: 10rem;
  width: 10rem;

}

tr {
  opacity: 0.8;
}

.node {
  height: 2rem;
  width: 2rem;
  border: 3px solid var(--bg-color);
  border-radius: 50%;
  display: inline-block;
  box-sizing: border-box;
}

/*.node-placeholder {*/
/*  width: 100%;*/
/*  height: 100%;*/
/*  display: flex;*/
/*  flex-direction: column;*/
/*}*/

/*.middle {*/
/*  display: flex;*/
/*  flex-direction: row;*/
/*}*/

/*.middle:first-child, .middle:last-child {*/
/*  width: 20%;*/
/*}*/

/*.node-placeholder div:nth-child(2) {*/
/*  width: 2rem;*/
/*}*/

/*.node-placeholder div:first-child, .div:last-child{*/
/*  flex: 1;*/
/*}*/
/*.top .arrow{*/
/*  transform: rotate(180deg);*/
/*}*/
/*.hidden {*/
/*  visibility: hidden;*/
/*}*/

/*.cross-line {*/
/*  height: 100%;*/
/*  width: 100%;*/
/*  display: flex;*/
/*  flex-wrap: wrap;*/
/*}*/

/*.cross-line div {*/
/*  width: 50%;*/
/*  height: 50%;*/
/*  box-sizing: border-box;*/
/*}*/

/*.cross-line div:first-child {*/
/*  border-right: black 1px solid;*/
/*  border-bottom: black 1px solid;*/
/*}*/

/*.cross-line div:last-child {*/
/*  border-left: black 1px solid;*/
/*  border-top: black 1px solid;*/
/*}*/

/*.arrow {*/
/*  height: auto;*/
/*  width: auto;*/
/*}*/

/*.corner {*/
/*  width: 50%;*/
/*  height: 50%;*/
/*  margin-top: calc(50% - 1px);*/
/*  box-sizing: border-box;*/
/*  border-top: black 1px solid;*/
/*  border-right: black 1px solid;*/
/*}*/

/*.revert-corner {*/
/*  width: 50%;*/
/*  height: 50%;*/
/*  margin-left: 50%;*/
/*  margin-bottom: 50%;*/
/*  box-sizing: border-box;*/
/*  border-left: black 1px solid;*/
/*  border-bottom: black 1px solid;*/
/*}*/

/*.line {*/
/*  height: 100%;*/
/*  width: 100%;*/
/*  display: flex;*/
/*}*/

/*.vertical-line {*/
/*  flex-direction: row;*/
/*  box-sizing: content-box;*/
/*}*/

/*.horizontal-line {*/
/*  flex-direction: column;*/
/*}*/

/*.vertical-line div, .horizontal-line div {*/
/*  flex: 1;*/
/*}*/

/*.vertical-line div:last-child {*/
/*  height: 100%;*/
/*  width: 100%;*/
/*  border-left: black 1px solid;*/
/*}*/

/*.horizontal-line div:first-child {*/
/*  height: 100%;*/
/*  width: 100%;*/
/*  border-bottom: black 1px solid;*/
/*}*/

</style>
