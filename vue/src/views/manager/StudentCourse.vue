<template>
  <div>
    <div class="card" style="margin-bottom: 10px;">
      <el-input  style="width: 265px;margin-left: 10px;" v-model="data.name" :prefix-icon="Search" placeholder="请输入课程名称"/>
      <el-input  style="width: 265px;margin-left: 10px;" v-model="data.no" :prefix-icon="Search" placeholder="请输入课程编号"/>
      <el-button type="primary" style="margin-left: 10px;" @click="load">查询</el-button>
      <el-button type="info" @click="reset">重置</el-button>
    </div>
    <div class="card" style="margin-bottom: 10px;">
      <div>
        <el-table :data="data.tableData" style="width: 100%">
          <el-table-column prop="id" label="序号" width="72" />
          <el-table-column prop="name" label="课程名称" />
          <el-table-column prop="no" label="课程编号" />
          <el-table-column prop="studentName" label="学生名称" />
          <el-table-column label="操作">
            <!--            作用域插槽-->
            <template #default = "scope">
              <el-button type="danger" @click="del(scope.row.id)">删除选课</el-button>
              <el-button type="primary" @click="addGrade(scope.row)" v-if="data.user.role === 'ADMIN'">成绩打分</el-button>
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
    <el-dialog width="35%" v-model="data.formVisible" title="成绩信息">
      <el-form :model="data.gradeForm" label-position="right" style="padding-right: 50px">
        <el-form-item label="课程名称:" label-width="100px">
          <el-input v-model="data.gradeForm.name" autocomplete="off" disabled/>
        </el-form-item>
        <el-form-item label="分数:" label-width="100px">
          <el-input v-model="data.gradeForm.score" autocomplete="off" />
        </el-form-item>
        <el-form-item label="评语:" label-width="100px">
          <el-input type="textarea" v-model="data.gradeForm.comment" autocomplete="off" />
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
  tableData:[],
  total:0,
  pageNum:1,//当前的页码
  pageSize:3,//每页的个数
  user: JSON.parse(localStorage.getItem('student-user') || '{}'),
  gradeForm:{},
  formVisible:false
})
onMounted(() => {
  load()
});
//查询分页数据
const load = ()=>{
  let  params = {
    pageNum:data.pageNum,
    pageSize:data.pageSize,
    name:data.name,
    no:data.no,
    student_id:''
  }
  if (data.user.role === "STUDENT"){//如果当前登录的是学生，那就查询自己的选课记录
   params.student_id = data.user.id;//data.user.id当前登陆者的id
  }
  request.get('/studentCourse/selectPage',{
    params: params
  }).then(res =>{
    data.tableData = res.data?.list || [];
    data.total = res.data?.total || 0;
  })
}
//重置
const reset = () => {
  data.name =''
  data.no =''
  load()
}
const handelCurrentChange = (pageNum)=> {//点击的当前页码
  load();//当换页的时候重新加载数据
}
//删除选课
const del = (id)=> {
  ElMessageBox.confirm("确认要删除吗","确认删除",{type:'warning'}).then(res =>{
    request.delete('/studentCourse/delete/' + id)
        .then(res => {
          if(res.code === '200'){
            //重新获取数据
            load();
            ElMessage.success("删除选课成功")
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
//成绩打分
const addGrade = (row) => {//当前行对象
  console.log(data.tableData,666)
  console.log(row,555)
  //弹窗
  data.formVisible = true;
  data.gradeForm.name = row.name
  data.gradeForm.studentId = row.studentId;//学生的id
  data.gradeForm.courseId = row.courseId;//课程的id
}
const save = () => {
  request.post('/grade/add',data.gradeForm).then(
      res =>{
        if(res.code === '200'){
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
</script>