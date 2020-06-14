/**
 * 基础菜单 商品管理
 */
<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索筛选 -->
    <el-form :inline="true" :model="searchModel" class="user-search">
      <el-form-item label="搜索：">
        <el-input size="small" v-model="searchModel.title" placeholder="请输入商品名称"></el-input>
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
      <el-table-column
        prop="title"
        label="商品名称"
        width="280">
      </el-table-column>
      <el-table-column
        prop="sellPrice"
        label="价格"
        width="100">
      </el-table-column>
      <el-table-column
        prop="updateTime"
        label="修改时间"
        width="280">
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
    <!-- 编辑界面 :data="form"-->
    <el-dialog :title="title" :visible.sync="editFormVisible" width="50%" @click="closeDialog">
      <el-form label-width="120px" :model="editForm" :rules="rules" ref="editForm">
        <el-form-item label="名称" prop="title">
          <el-input size="small" v-model="editForm.title" auto-complete="off" placeholder="请输入商品名称"></el-input>
        </el-form-item>
        <el-form-item label="货号" prop="goodsNo">
          <el-input size="small" v-model="editForm.goodsNo" auto-complete="off" placeholder="请输入货号"></el-input>
        </el-form-item>
        <el-form-item label="价格" prop="sellPrice">
          <el-input size="small" v-model="editForm.sellPrice" auto-complete="off" placeholder="请输入价格"></el-input>
        </el-form-item>
        <el-form-item label="摘要" prop="summary">
          <el-input type="textarea" v-model="editForm.summary" auto-complete="off" placeholder="请输入摘要"></el-input>
        </el-form-item>
        <el-form-item>
          <el-upload
            class="upload-demo"
            ref="upload"
            action="/api/fileUpload"
            :on-preview="handlePreview"
            :before-upload="beforeAvatarUpload"
            :on-remove="handleRemove"
            :file-list="fileList"
            :auto-upload = 'false'
            :on-success = 'handleSuccess'
            name="file">
            <el-button slot="trigger" size="small" type="primary">选取文件</el-button>
            <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件</div>
          </el-upload>
        </el-form-item>

        <el-form-item>
          <input type="hidden" v-model="editForm.imgUrl"/>
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
// import Pagination from '../../components/Pagination'
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
        id: '',
        title: '',
        goodsNo: '',
        sellPrice: '',
        summary: '',
        imgUrl: ''
      },
      // rules表单验证
      rules: {
        title: [{ required: true, message: '请输入名称', trigger: 'blur' }],
        sellPrice: [{ required: true, message: '请输入价格', trigger: 'blur' }],
        goodsNo: [{ required: true, message: '请输入货号', trigger: 'blur' }],
        summary: [{ required: true, message: '请输入摘要' }]
      },
      searchModel: {
        title: ''
      },
      // 删除
      seletedata: {
        ids: ''
      },
      fileName:'',
      fileList:[],
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
    // 获取列表
    getData() {
      this.loading = true
      this.$axios.get('/api/goods/pagelist',
        { params: {
            'curPage':this.curPage,
            'pageSize':this.pageSize,
            'title':this.searchModel.title
          },
          headers:{'X-USER-TOKEN':this.token}})
        .then((response) => {
          this.logining = false;
          //console.log(response.data);
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
    //显示编辑界面
    handleEdit: function(index, row) {
      console.log(row)
      this.editFormVisible = true
      this.title = '修改'
      if (row != undefined && row != 'undefined') {
        this.$axios.get('/api/goods/toEdit',{ params: {id: row.id},
          headers:{'X-USER-TOKEN':this.token}})
          .then((response) => {
            console.log(this);
            console.log(response.data);
            if(response.data.code==='0000'){
              this.editForm.id = row.id
              this.editForm.title = response.data.data.title
              this.editForm.goodsNo = response.data.data.goodsNo
              this.editForm.sellPrice = response.data.data.sellPrice
              this.editForm.summary = response.data.data.summary
              this.editForm.imgUrl = response.data.data.imgUrl

            }else{
              this.$message({
                message: response.data.msg,
                type: 'error'
              })
            }
          })
      } else {
        this.title = '添加'
        this.editForm.id = ''
        this.editForm.title = ''
        this.editForm.goodsNo = ''
        this.editForm.sellPrice = ''
        this.editForm.summary = ''
        this.editForm.imgUrl = ''
      }
    },
    // 编辑、增加页面保存方法
    submitForm(editData) {
      console.log(editData)
      this.$refs[editData].validate(valid => {
        if (valid) {
          this.$axios.defaults.headers.common["X-USER-TOKEN"] = this.token;
          this.$axios.post('/api/goods/saveOrUpdate', this.editForm)
            .then(res => {
              this.editFormVisible = false
              this.loading = false
              if (res.data.code=='0000') {
                this.getData()
                this.$message({
                  type: 'success',
                  message: '保存成功！'
                })
              } else {
                this.$message({
                  type: 'info',
                  message: res.data.msg
                })
              }
            })
            .catch(err => {
              this.editFormVisible = false
              this.loading = false
              this.$message.error('保存失败，请稍后再试！')
            })
        } else {
          return false
        }
      })
    },
    // 删除公司
    del(index, row) {
      this.$confirm('确定要删除吗?', '信息', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
          this.$axios.get('/api/goods/delete',{ params: {id: row.id},
            headers:{'X-USER-TOKEN':this.token}})
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
    },
    // 提交上传
    submitUpload() {
      this.$refs.upload.submit();
    },
    beforeAvatarUpload(file) {
      let img = file.name.split('.');
      if(img[1] === 'jpg'||img[1] === 'png'){
        return file
      }else {
        this.$message.error('上传文件只能是jpg/png格式!')
        return false
      }
    },
    handleRemove(file, fileList) {
    },
    handlePreview(file) {
    },
    handleSuccess(res,file,fileList){
      console.log(res)
      if (res.code=='0000') {
        console.log('==')
        this.editForm.imgUrl=res.data
        this.$message({
          message: '上传成功！',
          type: 'success'
        });
      } else {
        console.log('!=')
        this.$message({
          message: res.msg,
          type: 'error'
        });
      }

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


