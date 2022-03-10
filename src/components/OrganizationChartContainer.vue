<template>
  <div
    v-bind="{ scopedSlots: $slots }"
    class="relative inline-block border-2 border-dashed border-gray-400 rounded overflow-auto text-center"
    @wheel="zoom && zoomHandler($event)"
    @mouseup="pan && panning && panEndHandler($event)"
  >
    <div
      class="orgchart box-border inline-block touch-none select-none border border-dashed border-white p-5"
      :style="{
        transform: transformVal,
        cursor: cursorVal,
      }"
      @mousedown="pan && panStartHandler($event)"
      @mousemove="pan && panning && panHandler($event)"
    >
      <organization-chart-node
        :datasource="datasource"
        :handle-click="handleClick"
      >
        <template v-for="slot in Object.keys($slots)" v-slot:[slot]="scope">
          <slot :name="slot" v-bind="scope" />
        </template>
      </organization-chart-node>
    </div>
  </div>
</template>

<script>
import $ from "jquery";
import OrganizationChartNode from "./OrganizationChartNode.vue";

export default {
  name: "Container",
  props: {
    datasource: {
      type: Object,
      required: true,
    },
    pan: {
      type: Boolean,
      required: false,
    },
    zoom: {
      type: Boolean,
      required: false,
    },
    zoomoutLimit: {
      type: Number,
      required: false,
      default: 0.5,
    },
    zoominLimit: {
      type: Number,
      required: false,
      default: 7,
    },
  },
  data() {
    return {
      cursorVal: "default",
      panning: false,
      startX: 0,
      startY: 0,
      transformVal: "",
    };
  },
  components: {
    OrganizationChartNode,
  },
  methods: {
    handleClick(nodeData) {
      this.$emit("node-click", nodeData);
    },
    panEndHandler() {
      this.panning = false;
      this.cursorVal = "default";
    },
    panHandler(e) {
      let newX = 0;
      let newY = 0;
      if (!e.targetTouches) {
        // pand on desktop
        newX = e.pageX - this.startX;
        newY = e.pageY - this.startY;
      } else if (e.targetTouches.length === 1) {
        // pan on mobile device
        newX = e.targetTouches[0].pageX - this.startX;
        newY = e.targetTouches[0].pageY - this.startY;
      } else if (e.targetTouches.length > 1) {
        return;
      }
      if (this.transformVal === "") {
        if (this.transformVal.indexOf("3d") === -1) {
          this.transformVal = "matrix(1,0,0,1," + newX + "," + newY + ")";
        } else {
          this.transformVal =
            "matrix3d(1,0,0,0,0,1,0,0,0,0,1,0," + newX + ", " + newY + ",0,1)";
        }
      } else {
        let matrix = this.transformVal.split(",");
        if (this.transformVal.indexOf("3d") === -1) {
          matrix[4] = newX;
          matrix[5] = newY + ")";
        } else {
          matrix[12] = newX;
          matrix[13] = newY;
        }
        this.transformVal = matrix.join(",");
      }
    },
    panStartHandler(e) {
      if ($(e.target).closest(".node").length) {
        this.panning = false;
        return;
      } else {
        this.cursorVal = "move";
        this.panning = true;
      }
      let lastX = 0;
      let lastY = 0;
      if (this.transformVal !== "") {
        let matrix = this.transformVal.split(",");
        if (this.transformVal.indexOf("3d") === -1) {
          lastX = parseInt(matrix[4]);
          lastY = parseInt(matrix[5]);
        } else {
          lastX = parseInt(matrix[12]);
          lastY = parseInt(matrix[13]);
        }
      }
      if (!e.targetTouches) {
        // pand on desktop
        this.startX = e.pageX - lastX;
        this.startY = e.pageY - lastY;
      } else if (e.targetTouches.length === 1) {
        // pan on mobile device
        this.startX = e.targetTouches[0].pageX - lastX;
        this.startY = e.targetTouches[0].pageY - lastY;
      } else if (e.targetTouches.length > 1) {
        return;
      }
    },
    setChartScale(newScale) {
      let matrix = "";
      let targetScale = 1;
      if (this.transformVal === "") {
        this.transformVal =
          "matrix(" + newScale + ", 0, 0, " + newScale + ", 0, 0)";
      } else {
        matrix = this.transformVal.split(",");
        if (this.transformVal.indexOf("3d") === -1) {
          targetScale = Math.abs(window.parseFloat(matrix[3]) * newScale);
          if (
            targetScale > this.zoomoutLimit &&
            targetScale < this.zoominLimit
          ) {
            matrix[0] = "matrix(" + targetScale;
            matrix[3] = targetScale;
            this.transformVal = matrix.join(",");
          }
        } else {
          targetScale = Math.abs(window.parseFloat(matrix[5]) * newScale);
          if (
            targetScale > this.zoomoutLimit &&
            targetScale < this.zoominLimit
          ) {
            matrix[0] = "matrix3d(" + targetScale;
            matrix[5] = targetScale;
            this.transformVal = matrix.join(",");
          }
        }
      }
    },
    zoomHandler(e) {
      let newScale = 1 + (e.deltaY > 0 ? -0.2 : 0.2);
      this.setChartScale(newScale);
    },
  },
};
// import OrganizationChartNode from "./OrganizationChartNode.vue";
//
// export default {
//   name: "Container",
//   props: {
//     background: Boolean,
//     datasource: {
//       type: Object,
//       required: true,
//     },
//     pan: {
//       type: Boolean,
//       required: false,
//     },
//     zoom: {
//       type: Boolean,
//       required: false,
//     },
//     zoomOutLimit: {
//       type: Number,
//       required: false,
//       default: 0.5,
//     },
//     zoomInLimit: {
//       type: Number,
//       required: false,
//       default: 7,
//     },
//   },
//   data() {
//     return {
//       cursorVal: "default",
//       panning: false,
//       coordinate: {
//         start: {
//           x: 0,
//           y: 0,
//         },
//         current: {
//           x: 0,
//           y: 0,
//         },
//         last: {
//           x: 0,
//           y: 0,
//         },
//       },
//       transformVal: "",
//     };
//   },
//   components: {
//     OrganizationChartNode,
//   },
//   methods: {
//     handleClick(nodeData) {
//       this.$emit("node-click", nodeData);
//     },
//     panEndHandler() {
//       this.panning = false;
//       this.cursorVal = "default";
//     },
//     panHandler(event) {
//       if (!event.targetTouches) {
//         // pand on desktop
//         this.coordinate.current.x = event.pageX - this.coordinate.start.x;
//         this.coordinate.current.y = event.pageY - this.coordinate.start.y;
//       } else if (event.targetTouches.length === 1) {
//         // pan on mobile device
//         this.coordinate.current.x =
//           event.targetTouches[0].pageX - this.coordinate.start.x;
//         this.coordinate.current.y =
//           event.targetTouches[0].pageY - this.coordinate.start.y;
//       } else if (event.targetTouches.length > 1) {
//         return;
//       }
//
//       if (this.transformVal !== "") {
//         let matrix = this.transformVal.split(",");
//         if (this.transformVal.indexOf("3d") === -1) {
//           matrix[4] = this.coordinate.current.x;
//           matrix[5] = this.coordinate.current.y + ")";
//         } else {
//           matrix[12] = this.coordinate.current.x;
//           matrix[13] = this.coordinate.current.y;
//         }
//         this.transformVal = matrix.join(",");
//       }
//
//       if (this.transformVal.indexOf("3d") !== -1) {
//         return (this.transformVal =
//           "matrix3d(1,0,0,0,0,1,0,0,0,0,1,0," +
//           this.coordinate.current.x +
//           ", " +
//           this.coordinate.current.y +
//           ",0,1)");
//       }
//       return (this.transformVal =
//         "matrix(1,0,0,1," +
//         this.coordinate.current.x +
//         "," +
//         this.coordinate.current.y +
//         ")");
//     },
//     panStartHandler(event) {
//       this.cursorVal = "move";
//       this.panning = true;
//       if (this.transformVal !== "") {
//         let matrix = this.transformVal.split(",");
//         if (this.transformVal.indexOf("3d") === -1) {
//           this.coordinate.last.x = parseInt(matrix[4]);
//           this.coordinate.last.y = parseInt(matrix[5]);
//         } else {
//           this.coordinate.last.x = parseInt(matrix[12]);
//           this.coordinate.last.y = parseInt(matrix[13]);
//         }
//       }
//       if (!event.targetTouches) {
//         // pand on desktop
//         this.coordinate.start.x = event.pageX - this.coordinate.last.x;
//         this.coordinate.start.y = event.pageY - this.coordinate.last.y;
//       } else if (event.targetTouches.length === 1) {
//         // pan on mobile device
//         this.coordinate.start.x =
//           event.targetTouches[0].pageX - this.coordinate.last.x;
//         this.coordinate.start.y =
//           event.targetTouches[0].pageY - this.coordinate.last.y;
//       }
//     },
//     setChartScale(newScale) {
//       let matrix = "";
//       let targetScale = 1;
//       if (this.transformVal === "") {
//         return (this.transformVal =
//           "matrix(" + newScale + ", 0, 0, " + newScale + ", 0, 0)");
//       }
//
//       matrix = this.transformVal.split(",");
//
//       if (this.transformVal.indexOf("3d") === -1) {
//         targetScale = Math.abs(window.parseFloat(matrix[3]) * newScale);
//         if (targetScale > this.zoomOutLimit && targetScale < this.zoomInLimit) {
//           matrix[0] = "matrix(" + targetScale;
//           matrix[3] = targetScale;
//           this.transformVal = matrix.join(",");
//         }
//         return;
//       }
//       targetScale = Math.abs(window.parseFloat(matrix[5]) * newScale);
//       if (targetScale > this.zoomOutLimit && targetScale < this.zoomInLimit) {
//         matrix[0] = "matrix3d(" + targetScale;
//         matrix[5] = targetScale;
//         this.transformVal = matrix.join(",");
//       }
//     },
//     zoomHandler(e) {
//       let newScale = 1 + (e.deltaY > 0 ? -0.2 : 0.2);
//       this.setChartScale(newScale);
//     },
//   },
// };
</script>

<style>
.orgchart table {
  border-spacing: 0;
}

.orgchart td {
  text-align: center;
  vertical-align: top;
  padding: 0;
}
</style>
