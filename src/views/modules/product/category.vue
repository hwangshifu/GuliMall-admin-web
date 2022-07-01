<!--  -->
<template>
  <el-tree :data="menus" :props="defaultProps"
           :expand-on-click-node="false"
  >
     <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            type="text"
            size="mini"
            @click="() => append(data)">
            添加
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            删除
          </el-button>
        </span>
      </span>
  </el-tree>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      menus: [],
      defaultProps: {
        children: 'children',
        label: 'name'
      }
    };
  },
  //方法集合
  methods: {
    // 获取菜单树
    listWithTree() {
      this.$http({
        url: this.$http.adornUrl('/product/category/list/tree'),
        method: 'get',
      }).then(({data}) => {
        console.log("所有的菜单树：", data)
        this.menus = data.data;
      })
    },
    append(data) {
      console.log("append：", data)
    },

    remove(node, data) {
      console.log("remove：node，data", node, data)

    },
    handleNodeClick(data) {
      console.log(data);
    }
  },
  created() {
    this.listWithTree();
  }
}
</script>
<style scoped>

</style>
