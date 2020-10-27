<template>
  <div class="mod-config">
    <el-dialog
  title="批量增加"
  :visible.sync="dialogVisible"
  width="30%"
  >
  <div v-for="(data,index) in batchAddForm" :key="index">
    <label>单词</label>
    <el-input v-model="data.name"></el-input>
    <label>翻译</label>
    <el-input v-model="data.chTranslate"></el-input>
  </div>
  <span slot="footer" class="dialog-footer">
    <el-button @click="addDomain()">添加</el-button>
    <el-button @click="removeDomain()" type="danger">移除</el-button>
    <el-button @click="dialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="submitBatchAddFrom()">确 定</el-button>
  </span>
</el-dialog>
    <el-form :inline="true" :model="dataForm" @keyup.enter.native="getDataList()">
      <el-form-item>
        <el-date-picker v-model="dateTime" type="date" placeholder="选择日期" value-format="yyyy-MM-dd">
    </el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button  type="primary" @click="addInToday()">当日新增</el-button>
        <el-button  type="danger" @click="deleteHandle()" :disabled="dataListSelections.length <= 0">批量删除</el-button>
        <el-button  type="primary" @click="batchAdd()">批量新增</el-button>
      </el-form-item>
    </el-form>
    <el-table
      :data="dataList"
      border
      v-loading="dataListLoading"
      @selection-change="selectionChangeHandle"
      style="width: 100%;">
      <el-table-column
        type="selection"
        header-align="center"
        align="center"
        width="50">
      </el-table-column>
      <el-table-column
        prop="id"
        header-align="center"
        align="center"
        label="">
      </el-table-column>
      <el-table-column
        prop="name"
        header-align="center"
        align="center"
        label="单词名称">
      </el-table-column>
      <el-table-column
        prop="chTranslate"
        header-align="center"
        align="center"
        label="中文翻译">
      </el-table-column>
      <el-table-column
        prop="createTime"
        header-align="center"
        align="center"
        label="添加时间">
      </el-table-column>
      <el-table-column
        prop="nums"
        header-align="center"
        align="center"
        label="出现次数">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button type="text" size="small" @click="addOrUpdateHandle(scope.row.id)">修改</el-button>
          <el-button type="text" size="small" @click="deleteHandle(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      @size-change="sizeChangeHandle"
      @current-change="currentChangeHandle"
      :current-page="pageIndex"
      :page-sizes="[10, 20, 50, 100]"
      :page-size="pageSize"
      :total="totalPage"
      layout="total, sizes, prev, pager, next, jumper">
    </el-pagination>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './english-add-or-update'
  export default {
    data () {
      return {
        dateTime:"",
        dialogVisible:false,
        dataForm: {
          key: ''
        },
        batchAddForm:[{
          id: 0,
          name: '',
          chTranslate: '',
          createTime: null,
          nums: 0
        }],
        dataList: [],
        pageIndex: 1,
        pageSize: 10,
        totalPage: 0,
        dataListLoading: false,
        dataListSelections: [],
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {

      removeDomain(){
        this.batchAddForm.pop()
      },
      submitBatchAddFrom(){
        this.dialogVisible=false
        this.$http({
          url: this.$http.adornUrl('/generator/english/batchSave'),
          method: 'post',
          data:this.$http.adornData(this.batchAddForm, false)
        }).then(({data})=>{
          if(data.code===0){
            this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                  this.batchAddForm=[{
                  id: 0,
                  name: '',
                  chTranslate: '',
                  createTime: null,
                  nums: 0
                  }]
                }
              })
          }
        })
      },
      addDomain(){
        this.batchAddForm.push({
           id: 0,
          name: '',
          chTranslate: '',
          createTime: null,
          nums: 0
        })
      },
      batchAdd(){
        this.dialogVisible=true
      },
      // 获取数据列表
      getDataList () {
        console.log(this.dateTime)
        this.dataListLoading = true
        
        this.$http({
          url: this.$http.adornUrl('/generator/english/list'),
          method: 'get',
          params: this.$http.adornParams({
            'page': this.pageIndex,
            'limit': this.pageSize,
            'dateTime': this.dateTime
          })
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page.list
            this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            this.totalPage = 0
          }
          this.dataListLoading = false
        })
      },
      // 每页数
      sizeChangeHandle (val) {
        this.pageSize = val
        this.pageIndex = 1
        this.getDataList()
      },
      // 当前页
      currentChangeHandle (val) {
        this.pageIndex = val
        this.getDataList()
      },
      // 多选
      selectionChangeHandle (val) {
        this.dataListSelections = val
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        var ids = id ? [id] : this.dataListSelections.map(item => {
          return item.id
        })
        this.$confirm(`确定对[id=${ids.join(',')}]进行[${id ? '删除' : '批量删除'}]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/generator/english/delete'),
            method: 'post',
            data: this.$http.adornData(ids, false)
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        })
      },
      addInToday(){
        this.$http({
            url: this.$http.adornUrl('/generator/english/addInToday'),
            method: 'get',
          }).then(({data})=>{
            if(data&&data.code===0){
              this.dataList = data.page
              console.log(data.page.length)
              this.totalPage = data.page.length
              this.pageSize=data.page.length
            }else{
              this.dataList=[]
              this.totalPage = 0
            }
          })
      }
    }
  }
</script>
