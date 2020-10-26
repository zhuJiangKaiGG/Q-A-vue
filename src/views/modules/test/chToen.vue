<template>
 <div>
     <el-row gutter="20">
         <el-col :span="8">
             <div class="grid-content"><el-tag>中文翻译</el-tag></div>
         </el-col>
         <el-col :span="8">
             <div class="grid-content"><el-tag>输入答案</el-tag></div>
         </el-col>
         <el-col :span="8">
             <div class="grid-content"><el-tag>正确答案</el-tag></div>
         </el-col>
     </el-row>
     <el-row gutter="20" v-for="(data,index) in dataList" :key="data.name">
         <el-col :span="8" >
             <div style="text-align:center"><el-tag >{{data.chTranslate}}</el-tag></div>
         </el-col>
         <el-col :span="8" >
             <div style="text-align:center"><el-input  placeholder="请翻译" v-model="count[index]"></el-input></div>
         </el-col>
         <el-col :span="8" >
             <div style="text-align:center"><el-button  type="primary" v-if="data.id" @click="checkAnswers(index)">查看答案</el-button></div>
             <div style="text-align:center"><el-tag v-if="!data.id">{{data.name}}</el-tag></div>
         </el-col>
     </el-row>
     <div class="change-button" @click="getDataList()"><el-button type="primary">换一组</el-button></div>
 </div>
 
</template>

<script>
 export default {
   data () {
     return {
        dataList:[],
        count:[],
        translate:'',
        checkAnswer:true
     }
   },
   activated() {
       this.getDataList()
   },
   methods: {
       checkAnswers (index) {
           this.dataList[index].id=0
       },
       getDataList () {
         this.count=[]
        this.$http({
          url: this.$http.adornUrl('/generator/english/randomselect'),
          method: 'get'
        }).then(({data}) => {
          if (data && data.code === 0) {
            this.dataList = data.page
            //this.totalPage = data.page.totalCount
          } else {
            this.dataList = []
            //this.totalPage = 0
          }
         // this.dataListLoading = false
        })
      }
   }
 }
</script>

<style  scoped>
.change-button{
    margin-left: 45%;
    margin-top: 10%;
}
.grid-content{
    background-color: lightblue;
    text-align: center;
}
  .el-row {
    margin-bottom: 20px;
  
  }

  .el-col {
    border-radius: 4px;
  }
  .bg-purple-dark {
    background: #99a9bf;
  }

  .bg-purple-light {
    background: #e5e9f2;
  }

  .row-bg {
    padding: 10px 0;
    background-color: #f9fafc;
  }
</style>
