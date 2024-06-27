<template>
  <div>
<div class="card" style="margin-bottom: 10px;">
  <el-input  style="width: 265px;margin-left: 10px;" v-model="data.name" :prefix-icon="Search" placeholder="请输入课程名称"/>
  <el-input  style="width: 265px;margin-left: 10px;" v-model="data.no" :prefix-icon="Search" placeholder="请输入课程编号"/>
  <el-input  style="width: 265px" v-model="data.teacher" :prefix-icon="Search" placeholder="请输入任课老师"/>
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
          <el-table-column prop="name" label="课程名称" />
          <el-table-column prop="no" label="课程编号" />
          <el-table-column prop="descr" label="课程描述" />
          <el-table-column prop="times" label="课时" />
          <el-table-column prop="teacher" label="任课老师" />
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
 <el-dialog width="35%" v-model="data.formVisible" title="课程信息">
      <el-form :model="data.form" label-position="right" style="padding-right: 50px">
        <el-form-item label="课程名称:" label-width="100px">
          <el-input v-model="data.form.name" autocomplete="off" />
        </el-form-item>
        <el-form-item label="课程编号:" label-width="100px">
          <el-input v-model="data.form.no" autocomplete="off" />
        </el-form-item>
        <el-form-item label="课程描述:" label-width="100px">
          <el-input v-model="data.form.descr" autocomplete="off" />
        </el-form-item>
        <el-form-item label="课时:" label-width="100px">
          <el-input v-model="data.form.times" autocomplete="off" />
        </el-form-item>
        <el-form-item label="任课老师:" label-width="100px">
          <el-input v-model="data.form.teacher" autocomplete="off" />
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
import {reactive,onMounted} from "vue";
import {Search} from "@element-plus/icons-vue";
import request from "@/utils/request";
import {ElMessage, ElMessageBox} from "element-plus";

const data = reactive({
  name:'',
  no:'',
  teacher:'',
  tableData:[],
  total:0,
  pageNum:1,//当前的页码
  pageSize:3,//每页的个数
  formVisible:false,//弹框默认不展示
  form:{
   id:'',name:'',no:'',descr:'',times:'',teacher:''
  }
})
onMounted(() => {
  load()
});
const load = ()=>{
  request.get('/course/selectPage',{
    params:{
      pageNum:data.pageNum,
      pageSize:data.pageSize,
      name:data.name,
      no:data.no,
      teacher:data.teacher
    }
  }).then(res =>{
    data.tableData = res.data?.list || [];
    data.total = res.data?.total || 0;
  })
}
//重置
const reset = () => {
      data.name =''
      data.teacher =''
      data.no =''
      load()
}
//换页
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
//发送请求，保存到后台数据库
  request.request({
    url: data.form.id ?'/course/update':'/course/add',
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
    request.delete('/course/delete/' + id)
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
</script>