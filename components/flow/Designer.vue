<template>
  <div class="w-full mt-1 flex px-1">
    <!--Node Types List..-->

    <!-- <FlowToolBar @addnode="drag" :nodeTypes="nodeTypes" class="w-[1/4]" /> -->

    <!--Drawing Board dined with div id :[drawflow_div] -->
    <div class="w-full bg-white">
      <div
        id="drawflow_div"
        ref="drawflow_div"
        @drop="drop($event)"
        @dragover="allowDrop($event)"
      ></div>
    </div>
    <!-- <div class="w-[300px] bg-red"></div> -->
  </div>
</template>

<script setup>
import Drawflow from "drawflow";
import styleDrawflow from "drawflow/dist/drawflow.min.css?inline";

import {
  onMounted,
  shallowRef,
  h,
  getCurrentInstance,
  render,
  readonly,
  ref,
} from "vue";
import ToolBarNode from "./ToolBarNode.vue";
import { propsToAttrMap } from "@vue/shared";
let isMounted = false;

const props = defineProps(["nodeTypes", "zoomLevel", "canAddOutput"]);
watch(
  () => props.canAddOutput,
  (nv, ov) => {
    if (isMounted && nv) {
      console.log("watching output");
      addOutput();
    }
  },
  { immediate: true }
);
function addOutput() {
  if (selectedNode.value) {
    console.log("Go Ahead,add output to selected node");
    editor.value.addNodeOutput(selectedNode.value.id);
  } else {
    console.log("cannot add node, selected node is null");
  }
}
watch(
  () => props.zoomLevel,
  (nv, ov) => {
    if (isMounted) {
      nv > ov ? editor.value.zoom_in() : editor.value.zoom_out();
    }
  },
  { immediate: true }
);
const nodeTypes = ref({});
watch(
  () => props.nodeTypes,
  (nv, ov) => {
    if (isMounted) {
      console.log("nodetpes set");
      nodeTypes.value = nv;
    }
  },
  { immediate: true }
);
function watchZomm() {
  editor.value.nextTick();
}

const editor = shallowRef({});
const internalInstance = getCurrentInstance();
const Vue = { version: 3, h, render };

internalInstance.appContext.app._context.config.globalProperties.$df = editor;

const dialogVisible = ref(false);
const dialogData = ref({});
const emits = defineEmits({});
onMounted(async () => {
  nodeTypes.value = props.nodeTypes;

  var elements = document.getElementsByClassName("drag-drawflow");
  for (var i = 0; i < elements.length; i++) {
    elements[i].addEventListener("touchend", drop, false);
    elements[i].addEventListener("touchmove", positionMobile, false);
    elements[i].addEventListener("touchstart", drag, false);
  }

  const id = document.getElementById("drawflow_div");
  editor.value = new Drawflow(
    id,
    Vue,
    internalInstance.appContext.app._context
  );
  editor.value.on("nodeSelected", function (id) {
    // flowData.value = "...node selected...";
    console.log("node selected is:-> " + id);
    let x = editor.value.getNodeFromId(id);
    console.log(x.name);

    // riase node selection event to the listener with ID of NODE >>
    // this unique id is assignedby drawflow, not business specific id
    emits("nodeSelected", { id: id, name: x.name });
    selectedNode.value = x;
  });

  editor.value.start();

  editor.value.registerNode("ToolBarNode", ToolBarNode, {}, {});

  addArrows(editor.value);

  //addNodeToDrawFlow(data, ev.clientX, ev.clientY);

  isMounted = true;
  await nextTick();
  editor.value.zoom_out();
  editor.value.zoom_out();
  editor.value.zoom_out();
  addNodeToDrawFlow_First();
  dontLetDeleteFirstNode(editor.value);
  attachDeleteConfirmation(editor.value);
});
function attachDeleteConfirmation(editor) {
  editor.removeNodeId = function (id) {
    if (id === "node-1") {
      return;
    } else {
      console.log("ABout to delete: " + id);
      var r = confirm("Are you sure to delete?");
      if (r == true) {
        this.removeConnectionNodeId(id);
        var moduleName = this.getModuleFromNodeId(id.slice(5));
        if (this.module === moduleName) {
          document.getElementById(id).remove();
        }
        delete this.drawflow.drawflow[moduleName].data[id.slice(5)];
        this.dispatch("nodeRemoved", id.slice(5));
      }
    }
  };
}

function dontLetDeleteFirstNode(editor) {
  editor.on("contextmenu", (e) => {
    const idNode = 1; // Example with nodeId ===  1
    if (editor.node_selected !== null) {
      if (editor.node_selected.id === `node-${idNode}`) {
        editor.node_selected = null;
      }
    }
  });
}

const selectedNode = ref({});
// function addArrows(editor) {
//   editor.createCurvature = function (
//     start_pos_x,
//     start_pos_y,
//     end_pos_x,
//     end_pos_y,
//     curvature_value,
//     type
//   ) {
//     const fix_arrow_distance = 15;
//     var line_x = start_pos_x;
//     var line_y = start_pos_y;
//     var x = end_pos_x;
//     var y = end_pos_y - fix_arrow_distance;
//     var curvature = curvature_value;
//     //type openclose open close other
//     let line_path = "";
//     switch (type) {
//       case "open":
//         if (start_pos_y >= end_pos_y) {
//           var hy1 = line_y + Math.abs(y - line_y) * curvature;
//           var hy2 = y - Math.abs(y - line_y) * (curvature * -1);
//         } else {
//           var hy1 = line_y + Math.abs(y - line_y) * curvature;
//           var hy2 = y - Math.abs(y - line_y) * curvature;
//         }
//         line_path +=
//           " M " +
//           line_x +
//           " " +
//           line_y +
//           " C " +
//           line_x +
//           " " +
//           hy1 +
//           " " +
//           x +
//           " " +
//           hy2 +
//           " " +
//           x +
//           "  " +
//           y;

//         break;
//       case "close":
//         if (start_pos_y >= end_pos_y) {
//           var hy1 = line_y + Math.abs(y - line_y) * (curvature * -1);
//           var hy2 = y - Math.abs(y - line_y) * curvature;
//         } else {
//           var hy1 = line_y + Math.abs(y - line_y) * curvature;
//           var hy2 = y - Math.abs(y - line_y) * curvature;
//         }
//         line_path +=
//           " M " +
//           line_x +
//           " " +
//           line_y +
//           " C " +
//           line_x +
//           " " +
//           hy1 +
//           " " +
//           x +
//           " " +
//           hy2 +
//           " " +
//           x +
//           "  " +
//           y;
//         break;
//       case "other":
//         if (start_pos_y >= end_pos_y) {
//           var hy1 = line_y + Math.abs(y - line_y) * (curvature * -1);
//           var hy2 = y - Math.abs(y - line_y) * (curvature * -1);
//         } else {
//           var hy1 = line_y + Math.abs(y - line_y) * curvature;
//           var hy2 = y - Math.abs(y - line_y) * curvature;
//         }
//         line_path +=
//           " M " +
//           line_x +
//           " " +
//           line_y +
//           " C " +
//           line_x +
//           " " +
//           hy1 +
//           " " +
//           x +
//           " " +
//           hy2 +
//           " " +
//           x +
//           "  " +
//           y;
//         break;
//       default:
//         var hy1 = line_y + Math.abs(y - line_y) * curvature;
//         var hy2 = y - Math.abs(y - line_y) * curvature;

//         line_path +=
//           " M " +
//           line_x +
//           " " +
//           line_y +
//           " C " +
//           line_x +
//           " " +
//           hy1 +
//           " " +
//           x +
//           " " +
//           hy2 +
//           " " +
//           x +
//           "  " +
//           y;
//     }
//     const arrow_path = ` M ${x} ${y} L ${x - 5} ${y - 8} L ${x + 5} ${
//       y - 8
//     } L ${x} ${y} Z`;
//     return line_path + arrow_path;
//   };
// }
function addArrows(editor) {
  editor.curvature = 1.0;
  // editor.node_style = "Vertical";
  editor.createCurvature = function (
    start_pos_x,
    start_pos_y,
    end_pos_x,
    end_pos_y,
    curvature_value,
    type
  ) {
    var line_x = start_pos_x;
    var line_y = start_pos_y;
    var x = end_pos_x;
    var y = end_pos_y;
    var curvature = curvature_value;
    //type openclose open close other
    switch (type) {
      case "open":
        if (start_pos_x >= end_pos_x) {
          var hx1 = line_x + Math.abs(x - line_x) * curvature;
          var hx2 = x - Math.abs(x - line_x) * (curvature * -1);
        } else {
          var hx1 = line_x + Math.abs(x - line_x) * curvature;
          var hx2 = x - Math.abs(x - line_x) * curvature;
        }
        return (
          " M " +
          line_x +
          " " +
          line_y +
          " C " +
          hx1 +
          " " +
          line_y +
          " " +
          hx2 +
          " " +
          y +
          " " +
          x +
          "  " +
          y
        );

        break;
      case "close":
        if (start_pos_x >= end_pos_x) {
          var hx1 = line_x + Math.abs(x - line_x) * (curvature * -1);
          var hx2 = x - Math.abs(x - line_x) * curvature;
        } else {
          var hx1 = line_x + Math.abs(x - line_x) * curvature;
          var hx2 = x - Math.abs(x - line_x) * curvature;
        } //M0 75H10L5 80L0 75Z

        return (
          " M " +
          line_x +
          " " +
          line_y +
          " C " +
          hx1 +
          " " +
          line_y +
          " " +
          hx2 +
          " " +
          y +
          " " +
          x +
          "  " +
          y +
          " M " +
          (x - 11) +
          " " +
          y +
          " L" +
          (x - 20) +
          " " +
          (y - 5) +
          "  L" +
          (x - 20) +
          " " +
          (y + 5) +
          "Z"
        );
        break;
      case "other":
        if (start_pos_x >= end_pos_x) {
          var hx1 = line_x + Math.abs(x - line_x) * (curvature * -1);
          var hx2 = x - Math.abs(x - line_x) * (curvature * -1);
        } else {
          var hx1 = line_x + Math.abs(x - line_x) * curvature;
          var hx2 = x - Math.abs(x - line_x) * curvature;
        }
        return (
          " M " +
          line_x +
          " " +
          line_y +
          " C " +
          hx1 +
          " " +
          line_y +
          " " +
          hx2 +
          " " +
          y +
          " " +
          x +
          "  " +
          y
        );
        break;
      default:
        var hx1 = line_x + Math.abs(x - line_x) * curvature;
        var hx2 = x - Math.abs(x - line_x) * curvature;

        //return ' M '+ line_x +' '+ line_y +' C '+ hx1 +' '+ line_y +' '+ hx2 +' ' + y +' ' + x +'  ' + y;
        return (
          " M " +
          line_x +
          " " +
          line_y +
          " C " +
          hx1 +
          " " +
          line_y +
          " " +
          hx2 +
          " " +
          y +
          " " +
          x +
          "  " +
          y +
          " M " +
          (x - 11) +
          " " +
          y +
          " L" +
          (x - 20) +
          " " +
          (y - 5) +
          "  L" +
          (x - 20) +
          " " +
          (y + 5) +
          "Z"
        );
    }
  };
}
// const flowData = defineProps({ data: String });
// to export data
function exportEditor() {
  dialogData.value = editor.value.export();
  dialogVisible.value = true;
}

const drag = (ev) => {
  console.log("toolbar : > " + ev);
  emits("addnode", ev);
};

const drop = (ev) => {
  if (ev.type === "touchend") {
    var parentdrawflow = document
      .elementFromPoint(
        mobile_last_move.touches[0].clientX,
        mobile_last_move.touches[0].clientY
      )
      .closest("#drawflow");
    if (parentdrawflow != null) {
      addNodeToDrawFlow(
        mobile_item_selec,
        mobile_last_move.touches[0].clientX,
        mobile_last_move.touches[0].clientY
      );
    }
    mobile_item_selec = "";
  } else {
    ev.preventDefault();
    var data = ev.dataTransfer.getData("node");

    addNodeToDrawFlow(data, ev.clientX, ev.clientY);
  }
};
const allowDrop = (ev) => {
  ev.preventDefault();
};

let mobile_item_selec = "";
let mobile_last_move = null;
function positionMobile(ev) {
  mobile_last_move = ev;
}
function addNodeToDrawFlow_First() {
  console.log("adding node by id: " + name);
  const nodeSelected = nodeTypes.value[0]; //.find((ele) => ele.code == name);

  let pos_x = -100;
  let pos_y = -100;

  editor.value.addNode(
    nodeSelected.name,
    nodeSelected.input,
    nodeSelected.output,
    pos_x,
    pos_y,
    name,
    {},
    "ToolBarNode",
    "vue"
  );
  var x = editor.value.export();
  emits("flowDataChanged", x);
}
function addNodeToDrawFlow(name, pos_x, pos_y) {
  console.log("adding node: " + name);
  pos_x =
    pos_x *
      (editor.value.precanvas.clientWidth /
        (editor.value.precanvas.clientWidth * editor.value.zoom)) -
    editor.value.precanvas.getBoundingClientRect().x *
      (editor.value.precanvas.clientWidth /
        (editor.value.precanvas.clientWidth * editor.value.zoom));
  pos_y =
    pos_y *
      (editor.value.precanvas.clientHeight /
        (editor.value.precanvas.clientHeight * editor.value.zoom)) -
    editor.value.precanvas.getBoundingClientRect().y *
      (editor.value.precanvas.clientHeight /
        (editor.value.precanvas.clientHeight * editor.value.zoom));

  const nodeSelected = nodeTypes.value.find((ele) => ele.code == name);
  editor.value.addNode(
    nodeSelected.name,
    nodeSelected.input,
    nodeSelected.output,
    pos_x,
    pos_y,
    name,
    {},
    "ToolBarNode",
    "vue"
  );
  var x = editor.value.export();
  console.log("X: " + x);
  emits("flowDataChanged", x);
}
</script>

<style scoped>
.header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid #494949;
}
.container {
  min-height: calc(100vh - 100px);
}
.column {
  border-right: 1px solid #494949;
}
.column ul {
  padding-inline-start: 0px;
  padding: 10px 10px;
}
.column li {
  background: white;
  /* transparent; */
}

.node {
  border-radius: 1px;
  border: 1px solid #494949;
  display: block;
  height: 60px;
  line-height: 40px;
  padding: 20px;
  margin: 10px 0px;
  cursor: move;
}
#drawflow_div {
  border: #494949 solid 1px;
  width: 100%;
  height: 690px;
  text-align: initial;
  background: white;
  background-size: 20px 20px;
  background-image: radial-gradient(#dddddd 1px, transparent 1px);
}
</style>
