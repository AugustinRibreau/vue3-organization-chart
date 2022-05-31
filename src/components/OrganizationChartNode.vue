<template>
  <table class="table">
    <tbody>
      <tr>
        <td
          :colspan="
            datasource.children && datasource.children.length
              ? datasource.children.length * 2
              : null
          "
        >
          <div
            class="chartNode"
            style="min-width: 9rem"
            :id="datasource.id"
            @click.stop="handleClick(datasource)"
          >
            <slot :node-data="datasource">
              <div class="chartTitle">
                <p style="margin-left: 0.5rem; margin-right: 0.5rem">
                  {{ datasource.name }}
                </p>
              </div>
              <div class="chartContent">
                <p style="margin-left: 0.5rem; margin-right: 0.5rem">
                  {{ datasource.title }}
                </p>
              </div>
            </slot>
          </div>
        </td>
      </tr>
      <template v-if="datasource.children && datasource.children.length">
        <tr class="chartLines">
          <td :colspan="datasource.children.length * 2">
            <div class="chartDownLine"></div>
          </td>
        </tr>
        <tr class="chartLines">
          <td class="chartRightLine"></td>
          <template v-for="n in datasource.children.length - 1" v-bind:key="n">
            <td class="chartLeftLine chartTopLine"></td>
            <td class="chartRightLine chartTopLine"></td>
          </template>
          <td class="chartLeftLine"></td>
        </tr>
        <tr class="nodes">
          <td colspan="2" v-for="child in datasource.children" :key="child.id">
            <node :datasource="child" :handle-click="handleClick">
              <template
                v-for="slot in Object.keys($slots)"
                v-slot:[slot]="scope"
              >
                <slot :name="slot" v-bind="scope" />
              </template>
            </node>
          </td>
        </tr>
      </template>
    </tbody>
  </table>
</template>

<script>
export default {
  name: "node",
  props: {
    datasource: Object,
    handleClick: Function,
  },
  methods: {},
};
</script>

<style>
.table {
  margin-bottom: 0.5rem;
  margin-left: auto;
  margin-right: auto;
  border-collapse: separate;
}

.chartNode {
  box-sizing: border-box;
  display: inline-flex;
  flex-direction: column;
  position: relative;
  margin: 0 1px 2px 1px;
  max-width: 20px;
  border: 1px solid #f56868;
  text-align: center;
  transition-property: color, background-color, border-color,
    text-decoration-color, fill, stroke, opacity, box-shadow, transform, filter,
    backdrop-filter;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 300ms;
  border-radius: 0.35rem;
}
.chartNode:hover {
  box-shadow: 0 0 5px #f56868;
  cursor: default;
  z-index: 20;
}

.chartTitle {
  text-align: center;
  font-size: 0.75rem;
  font-weight: bold;
  line-height: 1.25rem;
  overflow: hidden;
  white-space: nowrap;
  background: #f56868;
  color: white;
  border-top-left-radius: 0.25rem;
  border-top-right-radius: 0.25rem;
}

.chartContent {
  box-sizing: border-box;
  width: 100%;
  height: 1.25rem;
  font-size: 0.75rem;
  line-height: 1rem;
  border: #f56868;
  text-align: center;
  border-bottom-right-radius: 0.25rem;
  border-bottom-left-radius: 0.25rem;
  background: white;
  color: black;
  overflow: hidden;
  white-space: nowrap;
}

.chartLines {
  height: 1.25rem;
}

.chartDownLine {
  background: #f56868;
  margin-left: auto;
  margin-right: auto;
  height: 1.25rem;
  width: 0.125rem;
  float: none;
}

.chartTopLine {
  border-top-color: #f56868;
  border-top-width: 2px;
}

.chartRightLine {
  border-right-color: #f56868;
  border-right-width: 1px;
}

.chartLeftLine {
  border-left-color: #f56868;
  border-left-width: 1px;
}
</style>
