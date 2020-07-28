<template>
    <div>home
    <el-button type="info" @click="edit">退出</el-button></div>
</template>

<script>
export default {
    methods:{
        edit:function(){
            window.sessionStorage.clear();
            this.$router.push('/login');
        }
    }
}
</script>
<style lang="less" scoped>

</style>

