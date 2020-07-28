<template>
    <div class="login_container">
       <div class="login_box">
           <!-- 头像区域 -->
           <div class="avatar_box">
               <img src="../assets/logo.png" alt="">
           </div>
           <!-- 登录表单区域 -->
           <el-form ref="loginFormRef" :model="loginForm" :rules="loginFormRules" label-width="0px" class="login_form">
               <!-- 用户名 -->
                <el-form-item prop="username">
                    <el-input v-model="loginForm.username" prefix-icon="iconfont icon-user"></el-input>
                </el-form-item>
                <!-- 密码 -->
                <el-form-item prop="password">
                    <el-input v-model="loginForm.password" prefix-icon="iconfont icon-3702mima" type="password"></el-input>
                </el-form-item>
                <!-- 按钮 -->
                <el-form-item class="btns">
                    <el-button type="success" @click="login">登录</el-button>
                    <el-button type="info" @click="resetLoginForm">重置</el-button>
                </el-form-item>
           </el-form>
       </div>
    </div>
</template>
<script>
export default {
    data(){
        return{
            //登录表单数据绑定对象
            loginForm:{
                username:'admin',
                password:'123456',
            },
            //表单验证规则对象
        loginFormRules:{
            // 验证用户名
            username:[{ required: true, message: '请输入账户名', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }],
            // 验证密码
            password:[{ required: true, message: '请输入密码', trigger: 'blur' },
            { min: 3, max: 10, message: '长度在 3 到 5 个字符', trigger: 'blur' }],
        }
        }
    },methods:{
        //重置按钮清除表单
        resetLoginForm:function(){
            this.$refs.loginFormRef.resetFields();
        },
        login:function(){
            //表单预验证
            this.$refs.loginFormRef.validate(async validate=>{
                // console.log(validate);
                if(!validate) return;
              const {data:res} =await  this.$http.post('login',this.loginForm);
              if(res.meta.status!=200) return this.$message.error('登录失败！');
              this.$message.success('登录成功!');
              console.log(res);
              window.sessionStorage.setItem('token',res.data.token);
              //编程式导航跳转后台
              this.$router.push('/home');
            }
            );
        }
    }
}
</script>
<style lang="less" scoped>
.login_container{
   background-color: rgb(215, 230, 230);
   height: 100%;
}
.login_box{
    width:450px;
    height: 300px;
    background-color: #fff;
    border-radius: 5px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%,-50%);
    .avatar_box{
        width: 130px;
        height: 130px;
        border: 1px solid #eee;
        border-radius: 50%;
        padding: 10px;
        box-shadow: 0 0 10px #ddd;
        position: absolute;
        left: 50%;
        transform: translate(-50%,-50%);
        background-color: #fff;
        img{
            width: 100%;
            height: 100%;
            border-radius: 50%;
            background-color: #eee;
        }
    }
}
.login_form{
    position: absolute;
    bottom: 0%;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;
}
.btns{
    display: flex;
    float: right;
}
</style>