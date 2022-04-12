<template>
  <div id="container">
    <v-stage
      :config="stageConfig"
      @mousewheel="mouseWheel"
      ref="stage_ref"
      @mousedown="mouseDownStage"
      @mousemove="mouseMoveStage"
      @mouseUp="mouseUpStage"
    >
      <v-layer ref="layer_ref">
        <v-line
          v-for="line in lines"
          :key="line.id"
          :config="line"
          :points="getPoints(getNodeById(line.from), getNodeById(line.to))"
        />
        <v-circle
          v-for="node in nodes"
          :key="node.id"
          :config="node"
          :draggable="dragState"
          @dragMove="updateNodeLocation"
          @mouseEnter="mouseEnterCircle"
          @mouseLeave="mouseLeaveCircle"
        />
      </v-layer>
    </v-stage>
  </div>
</template>
<script setup>
import Konva from "konva";
import { onMounted, reactive, ref } from "vue";
let stageConfig = {
  width: 600,
  height: 600,
};
let dragState = ref(true);

function generateCircle() {
  let num = 4;
  let res = [];
  while (res.length < num) {
    res.push({
      id: "circle_" + res.length,
      x: stageConfig.width * Math.random(),
      y: stageConfig.height * Math.random(),
      fill: Konva.Util.getRandomColor(),
      radius: 10 * Math.random() + 20,
      //把draggable状态改为响应式  draggable: dragstate.value没有响应只是赋值
    });
  }
  return res;
}
let nodes = reactive(generateCircle());
function generateLine() {
  let res = [];
  res.push({
    id: "line_1",
    stroke: "black",
    fill: "black",
    from: nodes[0].id,
    to: nodes[1].id,
  });
  return res;
}
let lines = reactive(generateLine());
function updateNodeLocation({ target }) {
  let node = getNodeById(target.attrs.id); //直接找到node对象
  node.x = target.attrs.x;
  node.y = target.attrs.y; //更新点的坐标
  //移动node后，该node位于图层最上方
  nodes.splice(nodes.indexOf(node), 1);
  nodes.push(node);
}

function getPoints(from, to) {
  //获取直线的所需的2个点的坐标
  const dx = to.x - from.x;
  const dy = to.y - from.y;
  let angle = Math.atan2(-dy, dx);
  return [
    from.x + -from.radius * Math.cos(angle + Math.PI),
    from.y + from.radius * Math.sin(angle + Math.PI),
    to.x + -to.radius * Math.cos(angle),
    to.y + to.radius * Math.sin(angle),
  ];
  // 这个长度正好不会出现：两圆重合后两直线戳出来
}
function getNodeById(id) {
  //通过id获得node的config ！！！非对象，只是一些属性
  return nodes.find((i) => i.id === id);
}
let layer_ref = ref(null),
  // eslint-disable-next-line no-unused-vars
  layer,
  stage_ref = ref(null),
  stage;
onMounted(() => {
  layer = layer_ref.value.getNode();
  stage = stage_ref.value.getNode();
});
function mouseWheel(e) {
  e.preventDefault(); //阻止默认事件
  let scaleBy = 1.05,
    oldScale = { x: stage.scaleX(), y: stage.scaleY() }; //原向量
  let pointer = stage.getPointerPosition();
  let mousePointTo = {
    x: (pointer.x - stage.x()) / oldScale.x,
    y: (pointer.y - stage.y()) / oldScale.y,
  }; //鼠标的实际位置
  let newScale;
  if (e.deltaY > 0) {
    newScale = {
      x: oldScale.x * scaleBy,
      y: oldScale.y * scaleBy,
    };
  } else if (e.deltaY < 0) {
    newScale = {
      x: oldScale.x / scaleBy,
      y: oldScale.y / scaleBy,
    };
  } //新的向量
  let newPos = {
    x: pointer.x - mousePointTo.x * newScale.x,
    y: pointer.y - mousePointTo.y * newScale.y,
  };
  stage.scale(newScale);
  stage.position(newPos);
}
function mouseEnterCircle() {
  stage.container().style.cursor = "pointer";
}
function mouseLeaveCircle() {
  stage.container().style.cursor = "default"; //因为圆在stage里面，所以离开圆时就是进入stage
}
let _mouseDownStage = false;
let oldStagePointer;
let oldStagePos;
function mouseDownStage(e) {
  if (e.target === stage) {
    _mouseDownStage = true;
    oldStagePointer = stage.getPointerPosition();
    oldStagePos = stage.position();
  }
}
function mouseMoveStage() {
  if (_mouseDownStage) {
    let pointer = stage.getPointerPosition();
    console.log(stage.x(), stage.y());
    stage.position({
      x: -oldStagePointer.x + pointer.x + oldStagePos.x,
      y: -oldStagePointer.y + pointer.y + oldStagePos.y, //偏移位置+stage位置
    });
  }
}
function mouseUpStage() {
  _mouseDownStage = false;
  // let pointer = stage.getPointerPosition();
}
</script>
<style>
#container {
  background-color: grey;
  height: 600px;
  width: 600px;
}
</style>
