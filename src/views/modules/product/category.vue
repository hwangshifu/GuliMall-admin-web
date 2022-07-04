<!--  -->
<template>
  <div>
    <el-tree
      show-checkbox
      node-key="catId"
      :data="menus" :props="defaultProps"
      :expand-on-click-node="false"
      draggable
      :allow-drop="allowDrop"
      :default-expanded-keys="expandedKeys">
     <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <=2"
            type="text"
            size="mini"
            @click="() => append(data)">
            添加
          </el-button>
          <el-button
            v-if="node.childNodes.length === 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)">
            删除
          </el-button>
          <el-button
            type="text"
            size="mini"
            @click="() => edit(data)">
            编辑
          </el-button>
        </span>
      </span>
    </el-tree>
    <el-dialog
      :title="titleName"
      :visible.sync="dialogVisible"
      :close-on-click-modal="false"
      width="30%">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input v-model="category.productUnit" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="submitData">确 定</el-button>
  </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  components: {},
  data() {
    return {
      // 弹框内容
      category: {
        maxLevel: 0,
        titleName: "", // 弹框提示信息
        dialogType: "",  // 区分编辑和新增的弹框
        catId: null,
        name: "",
        parentCid: "",
        catLevel: "",
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: ""
      },
      // 打开对话框
      dialogVisible: false,
      // 默认展开
      expandedKeys: [],
      // 菜单树
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

    // 获取界面新增信息
    append(data) {
      console.log("append：", data)
      this.dialogVisible = true
      this.category.parentCid = data.catId
      this.category.catLevel = data.catLevel * 1 + 1
      this.dialogType = "add"
      this.titleName = "新增菜单"
      this.category.icon = ""
      this.category.productUnit = ""
      this.category.catId = null
      this.category.sort = 0
      this.category.showStatus = 1
      this.category.name = ""
    },

    // 获取界面编辑信息
    edit(data) {
      console.log("更新的数据：", data)
      this.dialogVisible = true
      this.dialogType = "edit"
      this.titleName = "编辑菜单"
      // 获取详情
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: 'get',
        params: this.$http.adornParams({})
      }).then(({data}) => {
        this.category.name = data.data.name
        this.category.catId = data.data.catId
        this.category.icon = data.data.icon
        this.category.productUnit = data.data.productUnit
        this.category.parentCid = data.data.parentCid
      })
    },

    // 新增获取提交的信息
    addCategory() {
      console.log("提交的分类名称信息：", this.category)
      // 保存
      this.$http({
        url: this.$http.adornUrl('/product/category/save'),
        method: 'post',
        data: this.$http.adornData(this.category, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单添加成功',
          type: 'success'
        });
        // 关闭框
        this.dialogVisible = false
        //刷新列表
        this.listWithTree();
        // 设置默认展开的菜单
        this.expandedKeys = [this.category.parentCid]
      });
    },

    // 编辑获取提交的信息
    editCategory(data) {
      const {catId, name, icon, productUnit} = this.category;
      this.$http({
        url: this.$http.adornUrl('/product/category/update'),
        method: 'post',
        data: this.$http.adornData({catId, name, icon, productUnit}, false)
      }).then(({data}) => {
        this.$message({
          message: '菜单编辑成功',
          type: 'success'
        });
        // 关闭框
        this.dialogVisible = false
        //刷新列表
        this.listWithTree();
        // 设置默认展开的菜单
        this.expandedKeys = [this.category.parentCid]
      });
    },

    // 删除
    remove(node, data) {
      const ids = [data.catId];
      this.$confirm(`此操作将永久删除该${data.name}菜单, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$http({
          url: this.$http.adornUrl('/product/category/delete'),
          method: 'post',
          data: this.$http.adornData(ids, false)
        }).then(({data}) => {
          this.$message({
            message: '菜单删除成功',
            type: 'success'
          });
          //刷新列表
          this.listWithTree();
          // 设置默认展开的菜单
          this.expandedKeys = [node.parent.data.catId]
        });
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },

    submitData() {
      // 具体执行方法
      if (this.dialogType === "add") {
        this.addCategory()
      }
      if (this.dialogType === "edit") {
        this.editCategory()
      }
    },

    allowDrop(draggingNode, dropNode, type) {
      console.log("allowDrop：", draggingNode, dropNode, type)
      // 当前节点 和 父节点的深度不能大于3
      this.countNodeLevel(draggingNode);
      let deep = (this.maxLevel - draggingNode.data.catLevel) + 1
      console.log("当前节点深度：", deep)
      if (type === "inner") {
        return (deep + dropNode.level) <= 3
      } else {
        return (deep + dropNode.parent.level) <= 3
      }
    },

    countNodeLevel(node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes[i]);
        }
      }
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
