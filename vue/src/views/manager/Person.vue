<template>
<div>
  <div class="card" style="width: 50%; padding: 40px">
    <el-form :model="data.form" label-position="right" style="padding-right: 50px"  ref="formRef">
      <el-form-item label="头像:">
        <el-upload   class="avatar-uploader"   action="http://localhost:9090/files/upload" :show-file-list="false"
                     :on-success="handleImagUploadSuccess">
          <img v-if="data.form.avatar" :src="data.form.avatar" class="avatar"/>
          <el-icon v-else class="avatar-uploader-icon"><Plus /></el-icon>
        </el-upload>
      </el-form-item>

      <el-form-item label="学号:"  prop="username">
<!--        disabled 不允许设置-->
        <el-input v-model="data.form.username" autocomplete="off"  disabled/>
      </el-form-item>
      <el-form-item label="密码:" prop="password">
        <el-input show-password v-model="data.form.password" autocomplete="off" />
      </el-form-item>
      <el-form-item label="姓名:" >
        <el-input v-model="data.form.name" autocomplete="off" />
      </el-form-item>
      <el-form-item label="性别:" >
        <el-radio-group v-model="data.form.sex">
          <el-radio label="男"></el-radio>
          <el-radio label="女"></el-radio>
        </el-radio-group>
      </el-form-item>
      <el-form-item label="年龄:">
        <el-input v-model="data.form.age" autocomplete="off" />
      </el-form-item>
      <el-form-item label="生日:" >
        <el-date-picker format="YYYY-MM-DD" value-format="YYYY-MM-DD" v-model="data.form.birth"></el-date-picker>
      </el-form-item>
      <el-form-item label="手机号码:">
        <el-input v-model="data.form.phone" autocomplete="off" />
      </el-form-item>
      <el-form-item label="邮箱账号:" >
        <el-input v-model="data.form.email" autocomplete="off" />
      </el-form-item>
<el-form-item>
  <el-button type="primary" @click="update">保存</el-button>
</el-form-item>
    </el-form>
  </div>
</div>
</template>

<script setup>
import {reactive} from "vue";
import request from "@/utils/request";
import {ElMessage} from "element-plus";
import router from "@/router";
import {Plus} from "@element-plus/icons-vue";

const data = reactive({
  form: JSON.parse(localStorage.getItem('student-user') || "{}")
})
const handleImagUploadSuccess = (res)=> {
  data.form.avatar = res.data;
  console.log(res)
}
const update = () => {
  request.put('/student/update',data.form).then(res => {
    if(res.code === '200'){
      ElMessage.success("删除成功")
      router.push('/login')
    }
    else {
      ElMessage.error(res.msg)
    }
  })
}
</script>

<style>
.avatar-uploader .el-upload {
  border: 1px dashed var(--el-border-color);
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: var(--el-transition-duration-fast);
}

.avatar-uploader .el-upload:hover {
  border-color: var(--el-color-primary);
}

.el-icon.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 115px;
  height: 115px;
  text-align: center;
}
.avatar {
  width: 115px;
  height: 115px;
  display: block;
}
</style>

