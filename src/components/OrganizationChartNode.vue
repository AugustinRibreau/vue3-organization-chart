<template>
  <table class="mt-2 mb-2 mx-auto border-separate">
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
            class="chartNode border-box inline-block relative m-0 p-0.5 border-2 border-dashed border-transparent text-center transition duration-300 hover:bg-red-300 rounded hover:cursor-default hover:z-20"
            style="min-width: 9rem"
            :id="datasource.id"
            @click.stop="handleClick(datasource)"
          >
            <slot :node-data="datasource">
              <div
                class="text-center text-xs font-bold h-5 leading-5 overflow-hidden overflow-ellipsis whitespace-nowrap bg-red-600 text-white rounded-t"
              >
                <p class="mx-2 chartTitle">{{ datasource.name }}</p>
              </div>
              <div
                class="border-box w-full h-5 text-xs border border-red-600 text-center rounded-b bg-white text-black overflow-none overflow-ellipsis whitespace-nowrap"
              >
                <p class="mx-2 chartContent">{{ datasource.title }}</p>
              </div>
            </slot>
          </div>
        </td>
      </tr>
      <template v-if="datasource.children && datasource.children.length">
        <tr class="chartLines h-5">
          <td :colspan="datasource.children.length * 2">
            <div class="chartDownLine bg-red-400 mx-auto float-none h-5 w-0.5"></div>
          </td>
        </tr>
        <tr class="chartLines h-5">
          <td class="chartRightLine border-r border-red-400"></td>
          <template v-for="n in datasource.children.length - 1" v-bind:key="n">
            <td class="chartLeftLine chartTopLine border-l border-red-400 border-t-2"></td>
            <td class="chartRightLine chartTopLine border-r border-red-400 border-t-2"></td>
          </template>
          <td class="chartLeftLine border-l border-red-400"></td>
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
