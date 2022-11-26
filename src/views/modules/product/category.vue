<template>
  <div>
    <el-switch
      v-model="draggable"
      active-text="开启拖拽"
      inactive-text="关闭拖拽"
    >
    </el-switch>
    <el-button @click="batchSave" v-if="draggable">批量保存</el-button>
    <!-- 树形结构展示 -->
    <!-- :expand-on-click-node="false"表示在点击节点时不会收缩展开，只有点击前面小箭头的时候才展开收缩 -->
    <!-- show-checkbox  给节点加上复选框 -->
    <!-- node-key  表明节点唯一的属性 -->
    <el-tree
      :data="menus"
      :props="defaultProps"
      :expand-on-click-node="false"
      show-checkbox
      node-key="catId"
      :default-expanded-keys="expandedKey"
      @node-click="handleNodeClick"
      @node-drop="handleDrop"
      :draggable="draggable"
      :allow-drop="allowDrop"
    >
      <!-- node为当前节点，data为当前节点的数据 -->
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <!-- 三级菜单不能在添加子节点，所以加上if语句 -->
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
          >
            Append
          </el-button>
          <el-button type="text" size="mini" @click="() => edit(data)">
            Edit
          </el-button>
          <!-- 没有子节点才可以删除，所以加上if语句 -->
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
        </span>
      </span>
    </el-tree>
    <!-- 对话框（close-on-click-modal：点对话框外，对话框不消失） -->
    <el-dialog
      :title="title"
      :visible.sync="dialogVisible"
      width="30%"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
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
  name: "category",
  components: {},
  directives: {},
  data() {
    return {
      pCid: [],
      draggable: false,
      updateNodes: [],

      maxLevel: 0,
      // 对话框的title，是动态绑定的
      title: "",
      // 与表单双向绑定（里面的内容是添加数据时都需要有值的属性）
      category: {
        name: "",
        parentCid: 0,
        catLevel: 0,
        showStatus: 1,
        sort: 0,
        icon: "",
        productUnit: "",
        catId: null,
      },
      // 对话框默认关闭
      dialogVisible: false,
      // 对话框打开类型，是要修改还是要添加
      dialogType: "", //edit,add
      menus: [],
      // 删除后指定重新展开父节点
      expandedKey: [],
      // children表示哪个字段是他的子节点；label表示要显示哪个字段
      defaultProps: {
        children: "children",
        label: "name",
      },
    };
  },
  mounted() {},
  methods: {
    handleNodeClick(data) {
      console.log(data);
    },

    //批量保存
    batchSave() {
      this.$http({
        url: this.$http.adornUrl("/product/category/update/sort"),
        method: "post",
        data: this.$http.adornData(this.updateNodes, false),
      })
        .then(({ data }) => {
          this.$message({
            type: "success",
            message: "菜单顺序修改成功!",
          });
          // 刷新出新的菜单
          this.getMenus();
          // 设置需要默认展开的菜单
          this.expandedKey = this.pCid;
          // 清空数据
          this.updateNodes = [];
          this.maxLevel = 0;
          // this.pCid = 0;
        })
        .catch(() => {});
    },

    // 拖拽成功事件
    handleDrop(draggingNode, dropNode, dropType, ev) {
      console.log("handleDrop: ", draggingNode, dropNode, dropType);

      //1 当前节点最新的父节点
      let pCid = 0;
      let siblings = null;
      if (dropType == "before" || dropType == "after") {
        pCid =
          dropNode.parent.data.catId == undefined
            ? 0
            : dropNode.parent.data.catId;
        siblings = dropNode.parent.childNodes;
      } else {
        pCid = dropNode.data.catId;
        siblings = dropNode.childNodes;
      }
      this.pCid.push(pCid);
      //2 当前拖拽节点的最新顺序
      for (let i = 0; i < siblings.length; i++) {
        if (siblings[i].data.catId == draggingNode.data.catId) {
          // 如果遍历的是当前正在拖拽的节点
          let catLevel = draggingNode.level;
          if (siblings[i].level != draggingNode.level) {
            // 当前节点的层级发生变化
            catLevel = siblings[i].level;
            // 修改他子节点的层级
            this.updateChildNodeLevlel(siblings[i]);
          }
          this.updateNodes.push({
            catId: siblings[i].data.catId,
            sort: i,
            parentCid: pCid,
            catLevel: catLevel,
          });
        } else {
          this.updateNodes.push({ catId: siblings[i].data.catId, sort: i });
        }
      }
      //3 当前拖拽节点的最新层级（这部分内容转移到了批量保存方法里）
      console.log("updateNodes", this.updateNodes);
      
    },

    // 修改层级
    updateChildNodeLevlel(node) {
      if (node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          var cNode = node.childNodes[i].data;
          this.updateNodes.push({
            catId: cNode.catId,
            catLevel: node.childNodes[i].level,
          });
          this.updateChildNodeLevlel(node.childNodes[i]);
        }
      }
    },

    // 判断节点能否被拖拽（被拖动的当前节点以及所在的父节点总层数不能大于3）
    allowDrop(draggingNode, dropNode, type) {
      //1 计算被拖动的当前节点总层数
      console.log("allowDrop:", draggingNode, dropNode, type);
      var level = this.countNodeLevel(draggingNode);

      // 当前正在拖动的节点+父节点所在的深度不大于3即可
      let deep = Math.abs(this.maxLevel - draggingNode.level) + 1;
      console.log("深度:", deep);

      // 若拖到节点里面，深度是相加；其他时，深度是加上所在节点父节点的深度
      if (type == "innner") {
        return deep + dropNode.level <= 3;
      } else {
        return deep + dropNode.parent.level <= 3;
      }
    },
    // 计算当前节点的层数（找到所有子节点，求出最大深度）
    countNodeLevel(node) {
      if (node.childNodes != null && node.childNodes.length > 0) {
        for (let i = 0; i < node.childNodes.length; i++) {
          if (node.childNodes[i].level > this.maxLevel) {
            this.maxLevel = node.childNodes[i].level;
          }
          this.countNodeLevel(node.childNodes);
        }
      }
    },

    // 点击修改按钮
    edit(data) {
      console.log("要修改的数据", data);
      // 打开对话框
      this.dialogType = "edit";
      this.title = "修改分类";
      // 发送请求获取节点最新的数据,防止多人操作的时候没更新
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        // 请求成功
        console.log("要回显的数据", data);
        this.category.name = data.data.name;
        this.category.catId = data.data.catId;
        this.category.icon = data.data.icon;
        this.category.productUnit = data.data.productUnit;
        this.category.parentCid = data.data.parentCid;
        this.dialogVisible = true;
      });
    },

    // 修改分类菜单
    editCategory() {
      // 要修改的数据（这些都需要在edit()里回显）
      var { catId, name, icon, productUnit } = this.category;
      this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "post",
        data: this.$http.adornData({ catId, name, icon, productUnit }, false),
      })
        .then(({ data }) => {
          this.$message({
            type: "success",
            message: "菜单修改成功!",
          });
          // 关闭对话框
          this.dialogVisible = false;
          // 刷新出新的菜单
          this.getMenus();
          // 设置需要默认展开的菜单
          this.expandedKey = [this.category.parentCid];
        })
        .catch(() => {});
    },

    // 判断打开的哪个对话框
    submitData() {
      if (this.dialogType == "add") {
        this.addCategory();
      }
      if (this.dialogType == "edit") {
        this.editCategory();
      }
    },

    // 点击append，出现添加节点的对话框
    append(data) {
      console.log("append----", data);
      this.dialogType = "add";
      // 赋默认值，防止对话框内有原来的数据留存，不友好（乘1再加1是防止是个字符串，所以先转化为数字）
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      this.title = "添加分类";
      this.category.catId = null;
      this.category.name = null;
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
      // 打开对话框
      this.dialogVisible = true;
    },
    // 通过表单 添加分类
    // this.category是发送到后端的数据
    addCategory() {
      console.log("提交的三级分类数据", this.category);
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      })
        .then(({ data }) => {
          this.$message({
            type: "success",
            message: "菜单保存成功!",
          });
          // 关闭对话框
          this.dialogVisible = false;
          // 刷新出新的菜单
          this.getMenus();
          this.expandedKey = [this.category.parentCid];
        })
        .catch(() => {});
    },
    // 删除节点
    remove(node, data) {
      console.log("remove---", node);
      console.log("data---", data);
      // 把catId拼成数组
      var ids = [data.catId];
      // this.$confirm：弹窗，确认是否删除；若删除则调用.then()，若取消则调用.catch()（catch不能没有）
      this.$confirm(`是否删除【${data.name}】当前菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            // 请求的api
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          })
            .then(({ data }) => {
              this.$message({
                type: "success",
                message: "菜单删除成功!",
              });
              // 刷新出新的菜单
              this.getMenus();
              // 删除后重新展开父节点
              this.expandedKey = [node.parent.data.catId];
            })
            .catch(() => {});
        })
        .catch(() => {
          this.$message({
            type: "info",
            message: "已取消删除",
          });
        });
    },
    //   从后端获取数据，放入树形结构
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取到菜单数据：", data.data);
        this.menus = data.data;
      });
    },
  },
  // 生命周期-创建完成。可以访问当前this实例
  created() {
    this.getMenus();
  },
};
</script>
<style scoped>
</style>