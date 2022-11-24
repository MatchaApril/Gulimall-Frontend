<template>
<!-- 树形结构展示 -->
    <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
</template>

<script>
export default {
    name: 'category',
    components: {},
    directives: {},
     data() {
      return {
        data: [],
        defaultProps: {
          children: 'children',
          label: 'label'
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
        }).then(data=>{
            console.log("成功获取到菜单数据：",data)
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
