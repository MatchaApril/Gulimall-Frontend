<template>
<!-- 树形结构展示 -->
    <el-tree :data="menus" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
    name: 'category',
    components: {},
    directives: {},
     data() {
      return {
        menus: [],
        // children表示哪个字段是他的子节点；label表示要显示哪个字段
        defaultProps: {
          children: 'children',
          label: 'name'
        }
      };
    },
    mounted() {
        
    },
    methods: {
        handleNodeClick(data) {
        console.log(data);
      },
    //   从后端获取数据，放入树形结构
      getMenus(){
        this.$http({
          url: this.$http.adornUrl('/product/category/list/tree'),
          method: 'get'
        }).then(({data})=>{
            console.log("成功获取到菜单数据：",data.data)
            this.menus = data.data;
        })
      }
    },
    // 生命周期-创建完成。可以访问当前this实例
    created(){
        this.getMenus();
    }
};
</script>
<style scoped>
</style>