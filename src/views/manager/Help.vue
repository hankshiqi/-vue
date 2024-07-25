<template>
    <div>
        <div class="search">
            <el-input placeholder="请输入救援事项查询" style="width: 200px" v-model="name"></el-input>
            <el-select v-model="status" placeholder="请选择处理状态" style="width: 200px; margin-left: 20px;">
                <el-option value="待处理"></el-option>
                <el-option value="进行中"></el-option>
                <el-option value="已处理"></el-option>
            </el-select>
            <el-date-picker format="yyyy-MM-dd" value-format="yyyy-MM-dd" v-model="time"
                style="width: 200px; margin-left: 20px;">
            </el-date-picker>
            <el-button type="info" plain style="margin-left: 10px" @click="load(1)">查询</el-button>
            <el-button type="warning" plain style="margin-left: 10px" @click="reset">重置</el-button>
        </div>

        <div class="operation">
            <el-button type="primary" plain @click="handleAdd" v-if="user.role === 'USER'">我要救援</el-button>
            <el-button type="danger" plain @click="delBatch" v-if="user.role === 'ADMIN'">批量删除</el-button>
        </div>

        <div class="table">
            <el-table :data="tableData" strip @selection-change="handleSelectionChange">
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="id" label="序号" width="70" align="center" sortable></el-table-column>
                <el-table-column prop="name" label="救援事项"></el-table-column>
                <el-table-column prop="content" label="救援内容"></el-table-column>
                <el-table-column prop="type" label="救援类型"></el-table-column>
                <el-table-column prop="img" label="现场图片">
                    <template v-slot="scope">
                        <el-image v-if="scope.row.img" style="width: 50px" :src="scope.row.img"
                            :preview-src-list="[scope.row.img]"></el-image>
                    </template>
                </el-table-column>
                <el-table-column prop="location" label="事故地点"></el-table-column>
                <el-table-column prop="longitude" label="经度"></el-table-column>
                <el-table-column prop="latitude" label="纬度"></el-table-column>
                <el-table-column prop="time" label="救援时间"></el-table-column>
                <el-table-column prop="userId" label="救援者Id"></el-table-column>
                <el-table-column prop="phone" label="联系方式"></el-table-column>
                <el-table-column prop="status" label="处理状态">
                    <template v-slot="scope">
                        <el-tag type="danger" v-if="scope.row.status === '待处理'">待处理</el-tag>
                        <el-tag type="info" v-if="scope.row.status === '进行中'">进行中</el-tag>
                        <el-tag type="success" v-if="scope.row.status === '已处理'">已处理</el-tag>
                    </template>
                </el-table-column>
                <el-table-column prop="recordId" label="出警记录"></el-table-column>
                <el-table-column label="操作" align="center" width="180">
                    <template v-slot="scope">
                        <el-button size="mini" type="primary" plain @click="handleEdit(scope.row)"
                            v-if="user.role === 'USER' && scope.row.status === '待处理'">编辑</el-button>
                        <el-button size="mini" type="danger" plain @click="del(scope.row.id)">删除</el-button>
                        <el-button size="mini" type="primary" plain @click="handleRecord(scope.row)"
                            v-if="user.role === 'ADMIN' && scope.row.status === '待处理'">出警</el-button>
                    </template>
                </el-table-column>
            </el-table>

            <div class="pagination">
                <el-pagination background @current-change="handleCurrentChange" :current-page="pageNum"
                    :page-sizes="[5, 10, 20]" :page-size="pageSize" layout="total, prev, pager, next" :total="total">
                </el-pagination>
            </div>
        </div>

        <el-dialog title="救援信息" :visible.sync="fromVisible" width="40%" :close-on-click-modal="false" destroy-on-close>
            <el-form :model="form" label-width="100px" style="padding-right: 50px" :rules="rules" ref="formRef">
                <el-form-item label="救援事项" prop="name">
                    <el-input v-model="form.name" placeholder="救援事项"></el-input>
                </el-form-item>
                <el-form-item label="救援内容" prop="content">
                    <el-input type="textarea" v-model="form.content" placeholder="救援内容"></el-input>
                </el-form-item>
                <el-form-item label="救援类型" prop="type">
                    <el-select style="width: 100%" v-model="form.type">
                        <el-option v-for="item in ['火灾', '落水', '煤气中毒', '地震', '意外伤害', '交通事故', '其他']" :key="item"
                            :value="item" :label="item">
                        </el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="现场图片" prop="img">
                    <el-upload :action="$baseUrl + '/files/upload'" :headers="{ token: user.token }" list-type="picture"
                        :on-success="handleImgSuccess">
                        <el-button type="primary">上传</el-button>
                    </el-upload>
                </el-form-item>
                <el-form-item label="事故地点" prop="location">
                    <el-input v-model="form.location" placeholder="事故地点"></el-input>
                </el-form-item>
                <el-form-item label="经度" prop="longitude">
                    <div>
                        <el-input v-model="form.longitude" placeholder="事故地点" style="width: 80%;"></el-input>
                        <a href="https://api.map.baidu.com/lbsapi/getpoint/index.html"
                            style="margin-left: 5px;color: aqua;" target="_blank">选择坐标</a>
                    </div>
                </el-form-item>
                <el-form-item label="纬度" prop="latitude">
                    <el-input v-model="form.latitude" placeholder="纬度"></el-input>
                </el-form-item>
                <el-form-item label="联系方式" prop="phone">
                    <el-input v-model="form.phone" placeholder="联系方式"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="fromVisible = false">取 消</el-button>
                <el-button type="primary" @click="save">确 定</el-button>
            </div>
        </el-dialog>

        <el-dialog title="出警信息" :visible.sync="fromVisible1" width="40%" :close-on-click-modal="false" destroy-on-close>
            <el-form :model="form" label-width="100px" style="padding-right: 50px" :rules="rules" ref="formRef">
                <el-form-item label="救助员ID" prop="volId">
                    <el-select v-model="form.volId">
                        <el-option v-for="item in vols" :key="item.id" :value="item.id" :label="item.name"></el-option>
                    </el-select>
                </el-form-item>
                <el-form-item label="备注" prop="comment">
                    <el-input v-model="form.comment" placeholder="备注" type="textarea"></el-input>
                </el-form-item>
            </el-form>
            <div slot="footer" class="dialog-footer">
                <el-button @click="fromVisible1 = false">取 消</el-button>
                <el-button type="primary" @click="keepRecord">确 定</el-button>
            </div>
        </el-dialog>
    </div>
</template>
<script>
export default {
    name: "Help",
    data() {
        return {
            tableData: [],  // 所有的数据
            pageNum: 1,   // 当前的页码
            pageSize: 10,  // 每页显示的个数
            total: 0,
            name: null,
            status: null,
            time: null,
            fromVisible: false,
            form: {},
            user: JSON.parse(localStorage.getItem('xm-user') || '{}'),
            rules: {
            },
            ids: [],
            fromVisible1: false,
            vols: []
        }
    },
    created() {
        this.load(1)
    },
    methods: {
        keepRecord() {
            this.$request.post('/record/add', this.form).then(res => {
                if (res.code === '200') {  // 表示成功保存
                    this.$message.success('出警成功')
                    this.load(1)
                    this.fromVisible1 = false
                } else {
                    this.$message.error(res.msg)  // 弹出错误的信息
                }
            })
        },
        handleRecord(row) {
            this.form = { helpId: row.id }//注意这里的params的写法
            this.$request.get('/volunteer/selectAll', {params : { free: '是' }}).then(res => {
                this.vols = res.data || []
            })
            this.fromVisible1 = true
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
                        url: this.form.id ? '/help/update' : '/help/add',
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
            this.$confirm('您确定删除吗？', '确认删除', { type: "warning" }).then(response => {
                this.$request.delete('/help/delete/' + id).then(res => {
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
            this.$confirm('您确定批量删除这些数据吗？', '确认删除', { type: "warning" }).then(response => {
                this.$request.delete('/help/delete/batch', { data: this.ids }).then(res => {
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
            this.$request.get('/help/selectPage', {
                params: {
                    pageNum: this.pageNum,
                    pageSize: this.pageSize,
                    name: this.name,
                    status: this.status,
                    time: this.time
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
            this.time = null
            this.status = null
            this.load(1)
        },
        handleCurrentChange(pageNum) {
            this.load(pageNum)
        },
        handleImgSuccess(response, file, fileList) {
            this.form.img = response.data
        },
    }
}
</script>

<style scoped></style>