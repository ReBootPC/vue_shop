<template>
    <div>
        <!--面包屑导航区域-->
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>商品管理</el-breadcrumb-item>
            <el-breadcrumb-item>商品分类</el-breadcrumb-item>
        </el-breadcrumb>
        <!--卡片视图区域-->
        <el-card>
            <el-row>
                <el-col>
                    <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
                </el-col>
            </el-row>
            <!--表格区域-->
            <tree-table class="treeTable" :data="catelist" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border>
                <template slot="isOk" slot-scope="scope">
                    <i class="el-icon-success" v-if="scope.row.cat_deleted  === false" style="color: lightgreen;"></i>
                    <i class="el-icon-error" v-else style="color: red;"></i>
                </template>
                <template slot="order" slot-scope="scope">
                    <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
                    <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
                    <el-tag type="warning" size="mini" v-else>三级</el-tag>
                </template>
                <template slot="opt" slot-scope="scope">
                    <!--编辑功能未实现-->
                    <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.cat_id)">编辑</el-button>
                    <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeCateById(scope.row.cat_id)">删除</el-button>
                </template>
            </tree-table>
            <!--页码区域-->
            <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="queryInfo.pagenum" :page-sizes="[3, 5, 10, 15]" :page-size="queryInfo.pagesize" layout="total, sizes, prev, pager, next, jumper" :total="total"></el-pagination>
        </el-card>
        <!--添加分类对话框-->
        <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogClosed">
            <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px">
                <el-form-item label="分类名称：" prop="cat_name">
                    <el-input v-model="addCateForm.cat_name"></el-input>
                </el-form-item>
                <el-form-item label="父级分类：">
                    <el-cascader :options="parentCateList" :props="cascaderProps" v-model="selectedKeys" @change="parentCateChanged" clearable></el-cascader>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addCateDialogVisible = false">取消</el-button>
                <el-button type="primary" @click="addCate">确定</el-button>
            </span>
        </el-dialog>
        <!--修改分类对话框-->
        <el-dialog title="修改分类" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
                <el-form-item label="分类名称" prop="cat_name">
                    <el-input v-model="editForm.cat_name"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisible = false">取消</el-button>
                <el-button type="primary" @click="editCateInfo">确定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Cate",
        data() {
            return {
                queryInfo: {
                    type: 3,
                    pagenum: 1,
                    pagesize: 5
                },
                //商品分类的数据列表
                catelist: [],
                total: 0,
                columns: [
                    {
                        label: '分类名称',
                        prop: 'cat_name'
                    },
                    {
                        label: '是否有效',
                        type: 'template',
                        template: 'isOk'
                    },
                    {
                        label: '排序',
                        type: 'template',
                        template: 'order'
                    },
                    {
                        label: '操作',
                        type: 'template',
                        template: 'opt'
                    }
                ],
                addCateDialogVisible: false,
                addCateForm: {
                    cat_name: '',
                    cat_pid: 0,
                    cat_level: 0
                },
                addCateFormRules: {
                    cat_name: [
                        {
                            required: true,
                            message: '请输入分类名称',
                            trigger: 'blur'
                        }
                    ]
                },
                parentCateList: [],
                cascaderProps: {
                    expandTrigger: 'hover',
                    checkStrictly: true,
                    value: 'cat_id',
                    label: 'cat_name',
                    children: 'children'
                },
                selectedKeys: [],
                editDialogVisible: false,
                editForm: {},
                editFormRules: {
                    cat_name: [
                        {
                            required: true,
                            message: '请输入分类名称',
                            trigger: 'blur'
                        }
                    ]
                }
            }
        },
        created() {
            this.getCateList()
        },
        methods: {
            async getCateList() {
                //获取商品分类数据
                const { data: res } = await this.$http.get('categories', { params: this.queryInfo})
                if(res.meta.status !== 200) {
                    return this.$message.error('获取商品分类失败')
                }
                this.catelist = res.data.result
                this.total = res.data.total
            },
            handleSizeChange(newSize) {
                this.queryInfo.pagesize = newSize
                this.getCateList()
            },
            handleCurrentChange(newPage) {
                this.queryInfo.pagenum = newPage
                this.getCateList()
            },
            showAddCateDialog() {
                this.getParentCateList()
                this.addCateDialogVisible = true
            },
            async getParentCateList() {
                const { data: res } = await this.$http.get('categories', {params: {type: 2}})
                if(res.meta.status !== 200) {
                    return this.$message.error('获取父级分类失败')
                }
                this.parentCateList = res.data
            },
            parentCateChanged() {
                if (this.selectedKeys.length > 0) {
                    //父级分类的ID
                    this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
                    //为当前分类的等级赋值
                    this.addCateForm.cat_level = this.selectedKeys.length
                    return
                } else {
                    this.addCateForm.cat_pid = 0
                    this.addCateForm.cat_level = 0
                }
            },
            addCate() {
                this.$refs.addCateFormRef.validate(async valid => {
                    if (!valid) return
                    const { data: res } = await this.$http.post('categories', this.addCateForm)
                    if(res.meta.status !== 201) {
                        return this.$message.error('添加分类失败')
                    }
                    this.$message.success('添加分类成功')
                    this.getCateList()
                    this.addCateDialogVisible = false
                })
            },
            addCateDialogClosed() {
                this.$refs.addCateFormRef.resetFields()
                this.selectedKeys = []
                this.addCateForm.cat_level = 0
                this.addCateForm.cat_pid = 0
            },
            async removeCateById(id) {
                const confirmResult = await this.$confirm(
                    '此操作将永久删除该分类，是否继续？',
                    '提示',
                    {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning'
                    }
                ).catch(err => err)
                if(confirmResult !== 'confirm') {
                    return this.$message.info('已取消删除')
                }
                const { data: res } = await this.$http.delete('categories/' + id)
                if(res.meta.status !== 200) {
                    return this.$message.error('删除分类失败')
                }
                this.$message.success('删除分类成功')
                this.getCateList()
            },
            async showEditDialog(id) {
                const { data: res } = await this.$http.get('categories/' + id)
                if(res.meta.status !== 200) {
                    return this.$message.error('查询分类失败')
                }
                this.editForm = res.data
                this.editDialogVisible = true
            },
            editDialogClosed() {
                this.$refs.editFormRef.resetFields()
            },
            editCateInfo() {
                this.$refs.editFormRef.validate(async valid => {
                    if (!valid) return
                    const { data: res } = await this.$http.put('categories/' + this.editForm.cat_id, {cat_name: this.editForm.cat_name})
                    if(res.meta.status !== 200) {
                        return this.$message.error('更新分类名称失败')
                    }
                    this.editDialogVisible = false
                    this.getCateList()
                    this.$message.success('更新分类名称成功')
                })
            }
        }
    }
</script>

<style lang="less" scoped>
    .treeTable {
        margin-top: 15px;
    }
    .el-cascader {
        width: 100%;
    }
</style>