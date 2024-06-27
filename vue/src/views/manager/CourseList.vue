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
        <el-button type="primary" @click="selectCourse(scope.row)">选课</el-button>
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
  student: JSON.parse(localStorage.getItem('student-user') || '{}')
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

const reset = () => {
  data.name =''
  data.teacher =''
  data.no =''
  load()
}
const handelCurrentChange = (pageNum)=> {//点击的当前页码
  load();//当换页的时候重新加载数据
}
//选课
const selectCourse = (row) => {
  request.post('/studentCourse/add',{student_id: data.student.id,name: row.name,no: row.no,course_id: row.id})
      .then(res => {
        if (res.code === '200'){
          ElMessage.success("选课成功")
        }else {
          ElMessage.error(res.msg)
        }
      })
}
</script>