<script>
export default {
  name: "UserVolApply",
  created() {
    this.load()
  },
  methods:{

    handleCardImgSuccess(response) {
      this.form.cardImg = response.data
    },
    submit(){
      this.$request.post('/volApply/add' ,this.form).then(res => {
        if (res.code === '200') {
          this.$message.success('提交成功')
          this.load()
        } else {
          this.$message.error(res.msg)
        }
      })
    },
    load(){
      this.$request.get('/volApply/selectByUserId/'+this.user.id).then(res=>{
        this.form=res.data||{}
      })
    }

  },
  data(){
    return {
      user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
      form:{},
      rules:{}
    }
  }
}
</script>

<template>
<div style="width: 50%" class="card">
  <div style="text-align: center;font-size: 24px;margin-bottom: 20px"> 申请成为志愿者 </div>
  <el-form :model="form" label-width="100px" style="padding-right: 50px" :rules="rules" ref="formRef">
    <el-form-item label="电话" prop="phone">
      <el-input v-model="form.phone" placeholder="电话"></el-input>
    </el-form-item>
    <el-form-item label="性别" prop="sex">
      <el-radio-group v-model="form.sex">
        <el-radio label="男"></el-radio>
        <el-radio label="女"></el-radio>
      </el-radio-group>
    </el-form-item>
    <el-form-item label="生日" prop="birth">
      <el-date-picker format="yyyy-MM-dd" value-format="yyyy-MM-dd"
                      v-model="form.birth" style="width: 100%"></el-date-picker>
    </el-form-item>
    <el-form-item label="身份证号码" prop="cardNo">
      <el-input v-model="form.cardNo" placeholder="身份证号码"></el-input>
    </el-form-item>
    <el-form-item label="身份证照片" prop="cardImg">
      <el-upload
          :action="$baseUrl + '/files/upload'"
          :headers="{ token: user.token }"
          list-type="picture"
          :on-success="handleCardImgSuccess"
      >
        <el-button type="primary">上传个人照片</el-button>
      </el-upload>
    </el-form-item>
    <el-form-item label="个人介绍" prop="info">
      <el-input type="textarea" v-model="form.info" placeholder="个人介绍"></el-input>
    </el-form-item>
    <el-form-item label="审核状态" prop="status" v-if="form.status">
      <strong style="color: #409eff" v-if="form.status === '通过'">通过</strong>
      <strong style="color: #f56c6c" v-if="form.status === '拒绝'">拒绝</strong>
      <strong style="color: #909399" v-if="form.status==='待审核'">待审核</strong>
    </el-form-item>
    <div style="text-align: center">
      <el-button size="medium" type="primary" @click="submit">提交</el-button>
    </div>
  </el-form>
</div>
</template>

<style scoped>

</style>