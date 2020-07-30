<template>
    <el-container class="home-container">
        <!-- 头部区域 -->
        <el-header>
            <div>
                <img src="../assets/heima.png" alt="">
                <span>电商后台管理系统</span>
            </div>
            <el-button type="info" @click="edit">退出</el-button>
        </el-header>
        <!-- 主体区域 -->
        <el-container>
            <!-- 左侧区域 -->
        <el-aside :width="iscollapse ? '65px' : '200px'">
            <div class="toggle-button" @click="toggleCollapse">|||</div>
        <el-menu
      class="el-menu"
      background-color="rgb(223, 215, 200)"
      text-color="#130c0e"
      active-text-color="#7bbfea" unique-opened :collapse="iscollapse" :collapse-transition="false"
       :router="true" :default-active="activePath">
      <el-submenu  :index="item.id + ''" v-for="item in meunlist" :key='item.id'>
        <template slot="title">
          <i :class="iconsObj[item.id]"></i>
          <span>{{item.authName}}</span>
        </template>
          <el-menu-item :index="'/'+subItem.path" v-for="subItem in item.children" :key="subItem.id"
           @click="saveNavState('/'+subItem.path)">{{subItem.authName}}</el-menu-item>
      </el-submenu>
      </el-menu>
        </el-aside>
        <!-- 内容区域 -->
        <el-main>
            <!-- 路由占位符 -->
            <router-view></router-view>
        </el-main>
    </el-container>
    </el-container>
</template>

<script>
export default {
    data(){
        return{
            iscollapse:false,
            activePath:'',
            meunlist:[],
            iconsObj:{
                '125':'iconfont icon-user',
                '103':'iconfont icon-tijikongjian',
                '101':'iconfont icon-shangpin',
                '102':'iconfont icon-danju',
                '145':'iconfont icon-baobiao',
            }
        }
    },
    //初始化
    created(){
        this.getMenuList()
        this.activePath = window.sessionStorage.getItem('activePath')
    },
    methods:{
        edit:function(){
            window.sessionStorage.clear();
            this.$router.push('/login');
        },
        //获取所有菜单
        async getMenuList(){
            const {data:res} = await this.$http.get('menus')
            if(res.meta.status !== 200) return this.$message.error(res.meta.msg)
            this.meunlist = res.data
            // console.log(res);
        },
        toggleCollapse:function(){
            this.iscollapse= !this.iscollapse;
        },
        saveNavState(activePath){
            window.sessionStorage.setItem('activePath',activePath)
            this.activePath = activePath
        }
    }
}
</script>
<style lang="less" scoped>
.home-container{
    height: 100%;
}
.el-header{
    background-color: rgba(128, 128, 128, 0.801);
    display: flex;
    justify-content: space-between;
    align-items: center;
    padding-left: 0;
    color: #fff;
    size: 20px;
    >div{
        display: flex;
        align-items: center;
        span{
            margin-left: 15px;
        }
    }
}
.el-aside{
    background-color: rgb(223, 215, 218);
    .el-menu{
        border: 0px;
    }
}
.el-main{
    background-color: lightblue;
}
.iconfont{
    margin-right: 10px;
}
.toggle-button{
    background-color:rgb(223, 215, 200) ;
    color: #fff;
    font-size: 10px;
    line-height: 24px;
    text-align: center;
    cursor: pointer;
}
</style>