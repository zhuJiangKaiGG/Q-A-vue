<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
    <el-form-item label="题目名称" prop="name">
      <el-input v-model="dataForm.name" placeholder="题目名称"></el-input>
    </el-form-item>
    <el-form-item label="题目类型" prop="type">
      <el-input v-model="dataForm.type" placeholder="题目类型"></el-input>
    </el-form-item>
    <el-form-item label="题目出处" prop="provenance">
      <el-input v-model="dataForm.provenance" placeholder="题目出处"></el-input>
    </el-form-item>
    <el-form-item label="题目答案" prop="answer">
      <el-input v-model="dataForm.answer" placeholder="题目答案"></el-input>
    </el-form-item>
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
          type: '',
          provenance: '',
          answer: ''
        },
        dataRule: {
          name: [
            { required: true, message: '题目名称不能为空', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '题目类型不能为空', trigger: 'blur' }
          ],
          provenance: [
            { required: true, message: '题目出处不能为空', trigger: 'blur' }
          ],
          answer: [
            { required: true, message: '题目答案不能为空', trigger: 'blur' }
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
              url: this.$http.adornUrl(`/generator/programmer/info/${this.dataForm.id}`),
              method: 'get',
              params: this.$http.adornParams()
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.dataForm.name = data.programmer.name
                this.dataForm.type = data.programmer.type
                this.dataForm.provenance = data.programmer.provenance
                this.dataForm.answer = data.programmer.answer
              }
            })
          }
        })
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/generator/programmer/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'id': this.dataForm.id || undefined,
                'name': this.dataForm.name,
                'type': this.dataForm.type,
                'provenance': this.dataForm.provenance,
                'answer': this.dataForm.answer
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
