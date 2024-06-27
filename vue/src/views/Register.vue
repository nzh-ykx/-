<template>
  <div>
    <div class="login-container">
      <!--表单  -->
      <div style="width: 350px" class="login-box">
        <div style="font-weight: bold; font-size: 24px;text-align: center; margin-bottom: 30px;">注册</div>
        <el-form :model="data.form"  ref="formRef" :rules="rules">
          <el-form-item prop="username">
            <el-input  prefix-icon="User" v-model="data.form.username" placeholder="请输入账号"/>

          </el-form-item>
          <el-form-item prop="password">
            <el-input show-password prefix-icon="Lock" v-model="data.form.password" placeholder="请输入密码"/>
          </el-form-item>

          <el-form-item>
            <el-button type="primary" style="width: 100%" @click="register">注册</el-button>
          </el-form-item>

          <div style="margin-top: 30px; ">
            <div style="display: inline-block;text-align: left; width: 50%;" >
              已有账号请<a href="/">登录</a>
            </div>
            <div style="display: inline-block;text-align: right; width: 50%;">
              <a href="">忘记密码</a>
            </div>
          </div>

        </el-form>
      </div>
    </div>
  </div>
</template>

<script setup>
import {reactive,ref} from "vue";
import request from "@/utils/request";
import {ElMessage} from "element-plus";
import router from "@/router";
const data = reactive({
  form:{
    username:"",
    password:"",
  }
})
const rules = reactive({//自定义校验
  username: [
    { required: true, message: '请输入账号', trigger: 'blur' },
    // required:true表示为必须输入； message:""设置不符合校验规则时的提示信息；trigger:""设置校验的触发方式：‘change’:数据改变时触发；‘blur’:失去焦点时触发；
    { min: 1, max: 16, message: '请输入1位到16位的账号', trigger: 'blur' },
  ],
  password: [
    { required: true, message: '请输入密码', trigger: 'blur' },
    { min: 6, max: 16, message: '请输入6位到16位的密码', trigger: 'blur' },
  ]
})
const formRef = ref()
const  register = () => {
  formRef.value.validate((valid) =>{//validate验证方法,valid参数为布尔值，当所有的表单验证都通过时valid为true
    console.log(valid)
    if(valid){
      request.post('/register',data.form)//data.form:包括账号，密码和角色
          .then(res => {//发送请求
            if(res.code === '200'){
              ElMessage.success('注册成功')//消息弹窗提示
              router.push('/');//跳转到登录页
            }
            else {ElMessage.error(res.msg)}
          })
    }
  })
}
</script>

<style scoped>
/*overflow: hidden; 超出隐藏*/
.login-container{
  min-height: 100vh;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  background-image: url("@/assets/imgs/register.jpg");
  background-size:cover;
}
.login-box{
  background-color: rgba(255,255,255,.9);
  box-shadow: 0 0 10px rgba(0,0,0,0.1);
  padding: 30px;
  border-radius: 10px;
}
</style>