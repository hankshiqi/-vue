<template>
    <div>
      <div class="search">
        <el-input placeholder="请输入关键字查询" style="width: 200px" v-model="name"></el-input>
        <el-button type="info" plain style="margin-left: 10px" @click="load(1)">查询</el-button>
        <el-button type="warning" plain style="margin-left: 10px" @click="reset">重置</el-button>
      </div>
  
      <div class="operation">
        <el-button type="danger" plain @click="delBatch" v-if="user.role==='ADMIN'">批量删除</el-button>
      </div>
  
      <div class="table">
        <el-table :data="tableData" strip @selection-change="handleSelectionChange">
          <el-table-column type="selection" width="55" align="center" v-if="user.role==='ADMIN'"></el-table-column>
          <el-table-column prop="id" label="序号" width="70" align="center" sortable></el-table-column>
          <el-table-column prop="volId" label="救助员ID"></el-table-column>
          <el-table-column prop="status" label="状态"></el-table-column>
          <el-table-column prop="time" label="出警时间"></el-table-column>
          <el-table-column prop="helpId" label="事件ID"></el-table-column>
          <el-table-column prop="comment" label="备注"></el-table-column>
          <el-table-column label="操作" align="center" width="180">
            <template v-slot="scope">
              <el-button size="mini" type="primary" plain @click="HandleRe(scope.row,'救援中')" v-if="user.role==='Volunteer'">出警</el-button>
              <el-button size="mini" type="danger" plain @click="del(scope.row.id)" v-if="user.role==='ADMIN'">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
  
        <div class="pagination">
          <el-pagination
              background
              @current-change="handleCurrentChange"
              :current-page="pageNum"
              :page-sizes="[5, 10, 20]"
              :page-size="pageSize"
              layout="total, prev, pager, next"
              :total="total">
          </el-pagination>
        </div>
      </div>
  
      <el-dialog title="出警信息" :visible.sync="fromVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
        <el-form :model="form" label-width="100px" style="padding-right: 50px" :rules="rules" ref="formRef">
          <el-form-item label="救助员ID" prop="volId">
            <el-input v-model="form.volId" placeholder="救助员ID"></el-input>
          </el-form-item>
          <el-form-item label="状态" prop="status">
            <el-input v-model="form.status" placeholder="状态"></el-input>
          </el-form-item>
          <el-form-item label="出警时间" prop="time">
            <el-input v-model="form.time" placeholder="出警时间"></el-input>
          </el-form-item>
          <el-form-item label="事件ID" prop="helpId">
            <el-input v-model="form.helpId" placeholder="事件ID"></el-input>
          </el-form-item>
          <el-form-item label="备注" prop="comment">
            <el-input v-model="form.comment" placeholder="备注"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="fromVisible = false">取 消</el-button>
          <el-button type="primary" @click="save">确 定</el-button>
        </div>
      </el-dialog>
  
  
    </div>
  </template>
  <script>
  export default {
    name: "record",
    data() {
      return {
        tableData: [],  // 所有的数据
        pageNum: 1,   // 当前的页码
        pageSize: 10,  // 每页显示的个数
        total: 0,
        name: null,
        fromVisible: false,
        form: {},
        user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
        rules: {
        },
        ids: []
      }
    },
    created() {
      this.load(1)
    },
    methods: {
      HandleRe(row,status){
        this.$confirm('您确定要出警吗','确认出警',{type: 'warning'}).then(res=>{
            row.status=status,
            this.$request.put('/record/update',row).then(res => {
                if (res.code === '200') {  // 表示成功保存
                this.$message.success('处理成功')
                this.load(1)
              } else {
                this.$message.error(res.msg)  // 弹出错误的信息
              }
            })
        } 
        ).catch(err=>{});
      },
      handleAdd() {   // 新增数据
        this.form = {}  // 新增数据的时候清空数据
        this.fromVisible = true   // 打开弹窗
      },
      handleEdit(row) {   // 编辑数据
        this.form = JSON.parse(JSON.stringify(row))  // 给form对象赋值  注意要深拷贝数据
        this.fromVisible = true   // 打开弹窗
      },
      save() {   // 保存按钮触发的逻辑  它会触发新增或者更新
        this.$refs.formRef.validate((valid) => {
          if (valid) {
            this.$request({
              url: this.form.id ? '/record/update' : '/record/add',
              method: this.form.id ? 'PUT' : 'POST',
              data: this.form
            }).then(res => {
              if (res.code === '200') {  // 表示成功保存
                this.$message.success('保存成功')
                this.load(1)
                this.fromVisible = false
              } else {
                this.$message.error(res.msg)  // 弹出错误的信息
              }
            })
          }
        })
      },
      del(id) {   // 单个删除
        this.$confirm('您确定删除吗？', '确认删除', {type: "warning"}).then(response => {
          this.$request.delete('/record/delete/' + id).then(res => {
            if (res.code === '200') {   // 表示操作成功
              this.$message.success('操作成功')
              this.load(1)
            } else {
              this.$message.error(res.msg)  // 弹出错误的信息
            }
          })
        }).catch(() => {
        })
      },
      handleSelectionChange(rows) {   // 当前选中的所有的行数据
        this.ids = rows.map(v => v.id)
      },
      delBatch() {   // 批量删除
        if (!this.ids.length) {
          this.$message.warning('请选择数据')
          return
        }
        this.$confirm('您确定批量删除这些数据吗？', '确认删除', {type: "warning"}).then(response => {
          this.$request.delete('/record/delete/batch', {data: this.ids}).then(res => {
            if (res.code === '200') {   // 表示操作成功
              this.$message.success('操作成功')
              this.load(1)
            } else {
              this.$message.error(res.msg)  // 弹出错误的信息
            }
          })
        }).catch(() => {
        })
      },
      load(pageNum) {  // 分页查询
        if (pageNum) this.pageNum = pageNum
        this.$request.get('/record/selectPage', {
          params: {
            pageNum: this.pageNum,
            pageSize: this.pageSize,
            name: this.name,
          }
        }).then(res => {
          if (res.code === '200') {
            this.tableData = res.data?.list
            this.total = res.data?.total
          } else {
            this.$message.error(res.msg)
          }
        })
      },
      reset() {
        this.name = null
        this.load(1)
      },
      handleCurrentChange(pageNum) {
        this.load(pageNum)
      },
    }
  }
  </script>
  
  <style scoped>
  
  </style>