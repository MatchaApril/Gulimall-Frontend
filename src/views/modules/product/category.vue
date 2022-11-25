<template>
  <!-- 树形结构展示 -->
  <!-- :expand-on-click-node="false"表示在点击节点时不会收缩展开，只有点击前面小箭头的时候才展开收缩 -->
  <!-- show-checkbox  给节点加上复选框 -->
  <!-- node-key  表明节点唯一的属性 -->
  <el-tree :data="menus" :props="defaultProps" :expand-on-click-node="false" show-checkbox node-key="catId" @node-click="handleNodeClick">
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
</template>

<script>
export default {
  name: "category",
  components: {},
  directives: {},
  data() {
    return {
      menus: [],
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
    // 添加节点
    append(data) {
        console.log("append",data)
    },
    // 删除节点
    remove(node, data) {
         console.log("remove",node,data)
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