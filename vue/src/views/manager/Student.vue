<template>
  <div>
    <div class="card" style="margin-bottom: 10px;">
      <el-input  style="width: 265px;margin-left: 10px;" v-model="data.username" :prefix-icon="Search" placeholder="请输入账号查询"/>
      <el-input  style="width: 265px;margin-left: 10px;" v-model="data.name" :prefix-icon="Search" placeholder="请输入名称查询"/>
      <el-button type="primary" style="margin-left: 10px;" @click="load">查询</el-button>
      <el-button type="info" @click="reset">重置</el-button>
    </div>
    <div class="card" style="margin-bottom: 10px;">
      <div style="margin-bottom: 10px;">
        <el-button type="primary" @click="handleAdd">新增</el-button>
      </div>
      <div>
        <el-table :data="data.tableData" style="width: 100%">
          <el-table-column prop="id" label="序号" width="72" />
          <el-table-column prop="username" label="学号" />
          <el-table-column prop="name" label="姓名" />
          <el-table-column prop="sex" label="性别" />
          <el-table-column prop="age" label="年龄" />
          <el-table-column prop="birth" label="生日" />
          <el-table-column prop="phone" label="手机号码" />
          <el-table-column prop="email" label="邮箱账号" />
          <el-table-column prop="avatar" label="头像">
            <template #default="scope">
              <el-image v-if="scope.row.avatar" :src="scope.row.avatar"
                        :preview-src-list="[scope.row.avatar]" style="width: 40px;height: 40px;border-radius: 5px;"></el-image>
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <!--            作用域插槽-->
            <template #default = "scope">
              <el-button type="primary" @click="handleEdit(scope.row)">编辑</el-button>
              <!--              scope.row :scope是一个对象，把这一行作为一个参数传递过去-->
              <el-button type="danger" @click="del(scope.row.id)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
    <!--    分页器-->
    <div class="card">
      <el-pagination  v-model:current-page="data.pageNum" v-model:page-size="data.pageSize" background layout="prev, pager, next" :total="data.total"
                      @current-change="handelCurrentChange"
      />
    </div>
    <!--      //新增弹框-->
    <el-dialog width="35%" v-model="data.formVisible" title="学生信息" destroy-on-close>
<!--      destroy-on-close属性 弹窗关掉的时候把里面的元素全清空掉-->
      <el-form :model="data.form" label-position="right" style="padding-right: 50px" :rules="rules" ref="formRef">
        <el-form-item label="学号:"  prop="username">
          <el-input v-model="data.form.username" autocomplete="off" />
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
        <el-form-item label="头像:">
     <el-upload action="http://localhost:9090/files/upload" list-type="picture" :on-success="handleImagUploadSuccess">
      <el-button type="primary">上传头像</el-button>
     </el-upload>
        </el-form-item>
      </el-form>
      <template #footer>
      <span class="dialog-footer">
        <el-button @click="formVisible = false">取消</el-button>
        <el-button type="primary" @click="save">保存</el-button>
      </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import {ref,reactive,onMounted} from "vue";
import {Search} from "@element-plus/icons-vue";
import request from "@/utils/request";
import {ElMessage, ElMessageBox} from "element-plus";
const  baseUrl = '/student'
const formRef=ref()
const data = reactive({
  username:'',
  name:'',
  tableData:[],
  total:0,
  pageNum:1,//当前的页码
  pageSize:3,//每页的个数
  formVisible:false,//弹框默认不展示
  form:{
    id:'',username:'',password:'',name:'',sex:'',age:'',birth:'',phone:'',email:'',avatar:''
  }
})
onMounted(() => {
  load()
});
const load = ()=>{
  request.get(baseUrl+'/selectPage',{
    params:{
      pageNum:data.pageNum,
      pageSize:data.pageSize,
      username:data.username,
      name:data.name,
    }
  }).then(res =>{
    data.tableData = res.data?.list || [];
    data.total = res.data?.total || 0;
  })
}
const reset = () => {
  data.name =''
  data.username=''
  load()
}
//重置
const handelCurrentChange = (pageNum)=> {//点击的当前页码
  load();//当换页的时候重新加载数据
}
//新增
const handleAdd = ()=> {
  //清空上一次的残存数据
  data.form = {};
  //打开弹窗
  data.formVisible = true
}

//将新增和修改的数据保存到后台数据库
const  save = () => {
  formRef.value.validate((valid) =>{//validate验证方法,valid参数为布尔值，当所有的表单验证都通过时valid为true

    if(valid){
      request.request({
        url: data.form.id ? baseUrl+'/update': baseUrl+'/add',
        method: data.form.id ?'PUT':'POST',
        data: data.form
      })
          .then(res =>{
                if(res.code === '200'){
                  //重新获取数据
                  load();
                  //关闭弹窗
                  data.formVisible = false;
                  ElMessage.success("保存成功")
                }
                else {
                  ElMessage.error(res.msg)
                }
              }
          )
    }})

}
//修改编辑
const handleEdit = (row) => {
//深拷贝
  data.form  = JSON.parse(JSON.stringify(row));//把点击的这一行的数据拷贝到弹窗里面去
  data.formVisible = true;
}
//删除
const del = (id)=> {
  ElMessageBox.confirm("确认要删除吗","确认删除",{type:'warning'}).then(res =>{
    request.delete(baseUrl+'/delete/' + id)
        .then(res => {
          if(res.code === '200'){
            //重新获取数据
            load();
            ElMessage.success("删除成功")
          }
          else {
            ElMessage.error(res.msg)
          }
        })
  }) .catch(() => {
    ElMessage({
      type: 'info',
      message: 'Delete canceled',
    })
  })


}
//自定义校验
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
//头像上传
const handleImagUploadSuccess = (res)=> {
 data.form.avatar = res.data;
 console.log(res)
}
</script>