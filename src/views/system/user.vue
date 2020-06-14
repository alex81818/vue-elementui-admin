<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>系统用户管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索筛选 -->
    <el-form :inline="true" :model="searchModel" class="user-search">
      <el-form-item label="搜索：">
        <el-input size="small" v-model="searchModel.name" placeholder="请输入姓名"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button size="small" type="primary" icon="el-icon-search" @click="search">搜索</el-button>
        <el-button size="small" type="primary" icon="el-icon-plus" @click="handleEdit()">添加</el-button>
      </el-form-item>
    </el-form>
    <!--列表-->
    <el-table size="small" :data="listData" highlight-current-row v-loading="loading" border
              element-loading-text="拼命加载中" style="width: 100%;">
      <el-table-column align="center" type="selection" width="60" prop="id">
      </el-table-column>
      <el-table-column prop="username" label="用户名" width="100">
      </el-table-column>
      <el-table-column prop="name" label="姓名" width="100">
      </el-table-column>
      <el-table-column prop="email" label="邮箱" width="150">
      </el-table-column>
      <el-table-column prop="mobile" label="手机号" width="150">
      </el-table-column>
      <el-table-column prop="gmtModified" label="修改时间" width="150" :formatter="formatDate">
      </el-table-column>
      <el-table-column align="center" prop="status" label="禁用" min-width="150">
        <!--:active-value="0" :inactive-value="1"-->
        <!--<el-switch v-model="formatStatus" active-color="#ff4949" inactive-color="#13ce66"-->
          <!--active-text="正常" inactive-text="禁用" @change="changeStatus(scope.row, $event)">-->
        <!--</el-switch>-->
        <template slot-scope="scope">
          <el-switch
            v-model="scope.row.status"
            active-color="#ff4949"
            inactive-color="#13ce66"
            :active-value="1"
            :inactive-value="0"
            @change="changeStatus($event,scope.row,scope.$index)"
          />
        </template>
      </el-table-column>
      <el-table-column align="center" label="操作" min-width="300">
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button size="mini" type="danger" @click="del(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页组件 -->
    <!--<Pagination v-bind:child-msg="pageparm" @callFather="callFather"></Pagination>-->
    <el-pagination
      background
      layout="prev, pager, next"
      :total="total"
      :page-size="pageSize"
      @current-change="handleCurrentChange"
    >
    </el-pagination>
    <!-- 编辑界面 -->
    <el-dialog :title="title" :visible.sync="editFormVisible" width="50%" @click="closeDialog">
      <el-form label-width="120px" :model="editForm" :rules="rules" ref="editForm">
        <el-form-item label="用户名" prop="username">
          <el-input size="small" v-model="editForm.username" auto-complete="off" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="name">
          <el-input size="small" v-model="editForm.name" auto-complete="off" placeholder="请输入姓名"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input size="small" v-model="editForm.email" auto-complete="off" placeholder="请输入邮箱"></el-input>
        </el-form-item>
        <el-form-item label="手机号" prop="mobile">
          <el-input size="small" v-model="editForm.mobile" auto-complete="off" placeholder="请输入手机号"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click="closeDialog">取消</el-button>
        <el-button size="small" type="primary" :loading="loading" class="title" @click="submitForm('editForm')">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>
<script>
  import moment from 'moment'
  export default {
    data() {
      return {
        // 分页参数
        curPage: 1,
        pageSize: 10,
        total: 0,
        token: localStorage.getItem('logintoken'),
        nshow: true, //switch开启
        fshow: false, //switch关闭
        loading: false, //是显示加载
        editFormVisible: false, //控制编辑页面显示与隐藏
        title: '添加',
        editForm: {
          username: '',
          name: '',
          email: '',
          mobile: ''
        },
        // rules表单验证
        rules: {
          username: [{required: true, message: '请输入用户名', trigger: 'blur' }],
          name: [{required: true, message: '请输入姓名', trigger: 'blur'}],
          email: [
            {required: true, message: '请输入邮箱', trigger: 'blur' },
            {validator:function(rule,value,callback){
              if(/^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/.test(value) == false){
                callback(new Error("请输入正确的邮箱"));
              }else{
                callback();
              }
            }}],
          mobile: [
            {required: true, message: '请输入手机号', trigger: 'blur' },
            {validator:function(rule,value,callback){
                if(/^1[34578]\d{9}$/.test(value) == false){
                  callback(new Error("请输入正确的手机号"));
                }else{
                  callback();
                }
              }}]
        },
        searchModel: {
          name: ''
        },
        // 删除
        seletedata: {
          ids: ''
        },
        listData: [] //列表数据
      }
    },
    // 注册组件
    components: {
      //Pagination
    },
    /**
     * 创建完毕
     */
    created() {
      this.getData()
    },

    /**
     * 里面的方法只有被调用才会执行
     */
    methods: {
      changeStatus(e,row,index){        //e表示el-switch的状态（true，false）
        this.$axios.get('/api/sysuser/updateStatus',{params: {id: row.id, status: e ? 1 : 0},
          headers:{'X-USER-TOKEN':this.token}})
          .then(response => {
            console.log('切换状态成功');
            if(response.data.code==='0000'){
              this.$message({
                message: '操作成功',
                type: 'success'
              })
              this.getData()
            }else{
              this.$message({
                message: response.data.msg,
                type: 'error'
              })
            }
        }).catch(err => {
          console.log('切换状态失败');
        })
      },
      formatDate: function(row,column) {
        var newdate = row[column.property];
        if(newdate==undefined){return ''}
        return moment(newdate).format('YYYY-MM-DD HH:mm:ss')
      },
      // formatStatus: function(row) {
      //   return row.status == '0'
      // },
      //显示编辑界面
      handleEdit: function(index, row) {
        this.editFormVisible = true
        this.title = '修改'
        if (row != undefined && row != 'undefined') {
          this.$axios.get('/api/sysuser/toEdit',{params: {id: row.id},
            headers:{'X-USER-TOKEN':this.token}})
            .then((response) => {
              //console.log(response.data);
              if(response.data.code==='0000'){
                this.editForm.id=response.data.data.id
                this.editForm.username=response.data.data.username
                this.editForm.name=response.data.data.name
                this.editForm.email=response.data.data.email
                this.editForm.mobile=response.data.data.mobile
                this.editForm.status=response.data.data.status
                this.editForm.gmtModified=response.data.data.gmtModified
              }else{
                this.$message({
                  message: response.data.msg,
                  type: 'error'
                })
              }
            })
        } else {
          this.title = '添加'
          this.editForm.id=''
          this.editForm.username=''
          this.editForm.name=''
          this.editForm.email=''
          this.editForm.mobile=''
        }
      },

      // 获取列表
      getData() {
        console.log(moment().format());
        this.loading = true
        this.$axios.get('/api/sysuser/pagelist',{params: {'curPage':this.curPage,'pageSize':this.pageSize,'title':this.searchModel.name},headers:{'X-USER-TOKEN':this.token}})
          .then((response) => {
            this.logining = false;
            console.log(response.data);
            if(response.data.code==='0000'){
              this.loading = false
              this.listData = response.data.data.records
              this.total = response.data.data.total
              this.pageSize = response.data.data.size
              this.curPage = response.data.data.current
            }else{
              this.loading = false
              console.log(response.data);
              this.$message({
                message: response.data.msg,
                type: 'error'
              })
            }
          })
      },
      handleCurrentChange(val) {
        // 改变默认的页数
        this.curPage = val
        // 切换页码时，要获取每页显示的条数
        this.getData()
      },
      // 搜索事件
      search() {
        this.curPage=1
        this.getData(this.searchModel)
      },

      // 编辑、增加页面保存方法
      submitForm(editData) {
        console.log(editData)
        this.$refs[editData].validate(valid => {
          if (valid) {
            this.$axios.defaults.headers.common["X-USER-TOKEN"] = this.token;
            this.$axios.post('/api/sysuser/saveOrUpdate', this.editForm)
              .then(res => {
                if (res.data.code=='0000') {
                  this.editFormVisible = false
                  this.getData()
                  this.$message({
                    type: 'success',
                    message: '保存成功！'
                  })
                } else {
                  this.editFormVisible = true
                  this.$message({
                    type: 'error',
                    message: res.data.msg
                  })
                }
              })
              .catch(err => {
                this.editFormVisible = true
                this.loading = false
                this.$message.error('保存失败，请稍后再试！')
              })
          } else {
            return false
          }
        })
      },
      // 删除
      del(index, row) {
        this.$confirm('确定要删除吗?', '信息', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$axios.get('/api/sysuser/delete?id='+row.id)
            .then(res => {
              if (res.data.code=='0000') {
                this.$message({
                  type: 'success',
                  message: '删除成功!'
                })
                this.getData()
              } else {
                this.$message({
                  type: 'info',
                  message: res.msg
                })
              }
            })
            .catch(err => {
              this.loading = false
              this.$message.error('删除失败，请稍后再试！')
            })
        })
          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            })
          })
      },
      // 关闭编辑、增加弹出框
      closeDialog() {
        this.editFormVisible = false
      }
    }
  }
</script>

<style scoped>
  .user-search {
    margin-top: 20px;
  }
  .userRole {
    width: 100%;
  }
</style>




