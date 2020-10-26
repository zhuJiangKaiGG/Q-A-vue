<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="单词名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="单词名称"></el-input>
    </el-form-item>
    <el-form-item label="中文翻译" prop="chTranslate">
      <el-input v-model="dataForm.chTranslate" placeholder="中文翻译"></el-input>
    </el-form-item>
    <!-- <el-form-item label="添加时间" prop="createTime">
      <el-input v-model="dataForm.createTime" placeholder="添加时间"></el-input>
    </el-form-item>
    <el-form-item label="出现次数" prop="nums">
      <el-input v-model="dataForm.nums" placeholder="出现次数"></el-input>
    </el-form-item> -->
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  export default {
    data () {
      return {
        visible: false,
        dataForm: {
          id: 0,
          name: '',
          chTranslate: '',
          createTime: null,
          nums: 0
        },
        dataRule: {
          name: [
            { required: true, message: '单词名称不能为空', trigger: 'blur' }
          ],
          chTranslate: [
            { required: true, message: '中文翻译不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.visible = true
        this.$nextTick(() => {
          this.$refs['dataForm'].resetFields()
          if (this.dataForm.id) {
            this.$http({
              url: this.$http.adornUrl(`/generator/english/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.english.name
                this.dataForm.chTranslate = data.english.chTranslate
                this.dataForm.createTime = data.english.createTime
                this.dataForm.nums = data.english.nums
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            
            console.log(this.dataForm.createTime),
            this.$http({
              url: this.$http.adornUrl(`/generator/english/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'chTranslate': this.dataForm.chTranslate,
                'createTime': this.dataForm.createTime,
                'nums': this.dataForm.nums
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
