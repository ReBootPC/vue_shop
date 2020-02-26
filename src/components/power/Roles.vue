<template>
    <div>
        <el-breadcrumb separator-class="el-icon-arrow-right">
            <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
            <el-breadcrumb-item>权限管理</el-breadcrumb-item>
            <el-breadcrumb-item>角色列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!--卡片视图-->
        <el-card>
            <!--添加角色区域-->
            <el-row>
                <el-col>
                    <!--添加角色功能-->
                    <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
                </el-col>
            </el-row>
            <!--角色列表区域-->
            <el-table :data="rolelist" border stripe>
                <el-table-column type="expand">
                    <template slot-scope="scope">
                        <el-row :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']" v-for="(item1, i1) in scope.row.children" :key="item1.id">
                            <!--渲染一级权限-->
                            <el-col :span="5">
                                <el-tag closable @close="removeRightById(scope.row, item1.id)">
                                    {{item1.authName}}
                                </el-tag>
                                <i class="el-icon-caret-right">
                                </i>
                            </el-col>
                            <!--渲染二级权限-->
                            <el-col :span="19">
                                <el-row :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children" :key="item2.id">
                                    <el-col :span="6">
                                        <el-tag type="success" closable @close="removeRightById(scope.row, item2.id)">
                                            {{item2.authName}}
                                        </el-tag>
                                        <i class="el-icon-caret-right">
                                        </i>
                                    </el-col>
                                    <el-col :span="18">
                                        <el-tag v-for="(item3, i3) in item2.children" :key="item3.id" type="warning" closable @close="removeRightById(scope.row, item3.id)">
                                            {{item3.authName}}
                                        </el-tag>
                                    </el-col>
                                </el-row>
                            </el-col>
                        </el-row>
                    </template>
                </el-table-column>
                <el-table-column type="index">
                </el-table-column>
                <el-table-column label="角色名称" prop="roleName">
                </el-table-column>
                <el-table-column label="角色描述" prop="roleDesc">
                </el-table-column>
                <el-table-column label="操作" width="300px">
                    <template slot-scope="scope">
                        <!--编辑功能-->
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)">编辑</el-button>
                        <!--删除功能-->
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeRoleById(scope.row.id)">删除</el-button>
                        <el-button type="warning" icon="el-icon-setting" size="mini" @click="showSetRightDialog(scope.row)">分配权限</el-button>
                    </template>
                </el-table-column>
            </el-table>
        </el-card>
        <!--分配权限对话框-->
        <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
            <el-tree :data="rightslist" :props="treeProps" show-checkbox node-key="id" default-expand-all :default-checked-keys="defKeys" ref="treeRef"></el-tree>
            <span slot="footer" class="dialog-footer">
                <el-button @click="setRightDialogVisible = false">取 消</el-button>
                <el-button type="primary" @click="allotRights">确 定</el-button>
            </span>
        </el-dialog>
        <!--添加角色对话框-->
        <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
            <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="80px">
                <el-form-item label="角色名称" prop="roleName">
                    <el-input v-model="addForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc">
                    <el-input v-model="addForm.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="addDialogVisible = false">取消</el-button>
                <el-button type="primary" @click="addRole">确定</el-button>
            </span>
        </el-dialog>
        <!--修改角色对话框-->
        <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
                <el-form-item label="角色名称" prop="roleName">
                    <el-input v-model="editForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc">
                    <el-input v-model="editForm.roleDesc"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer"class="dialog-footer">
                <el-button @click="editDialogVisible = false">取消</el-button>
                <el-button type="primary" @click="editRoleInfo">确定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
    export default {
        name: "Roles",
        data() {
            return {
                rolelist: [],
                setRightDialogVisible: false,
                rightslist: [],
                treeProps: {
                    label: 'authName',
                    children: 'children'
                },
                defKeys: [],
                roleId: '',
                addDialogVisible: false,
                addForm: {
                    roleName: '',
                    roleDesc: ''
                },
                addFormRules: {
                    roleName: [
                        {
                            required: true,
                            message: '请输入角色名称',
                            trigger: 'blur'
                        },
                        {
                            min: 4,
                            max: 20,
                            message: '角色名称在4到20个字符之间',
                            trigger: 'blur'
                        }
                    ],
                    roleDesc: [
                        {
                            required: true,
                            message: '请输入角色描述',
                            trigger: 'blur'
                        },
                        {
                            min: 4,
                            max: 50,
                            message: '角色描述在4到50个字符之间',
                            trigger: 'blur'
                        }
                    ]
                },
                editDialogVisible: false,
                editForm: {},
                editFormRules: {
                    roleName: [
                        {
                            required: true,
                            message: '请输入角色名称',
                            trigger: 'blur'
                        },
                        {
                            min: 4,
                            max: 20,
                            message: '角色名称在4到20个字符之间',
                            trigger: 'blur'
                        }
                    ],
                    roleDesc: [
                        {
                            required: true,
                            message: '请输入角色描述',
                            trigger: 'blur'
                        },
                        {
                            min: 4,
                            max: 50,
                            message: '角色描述在4到50个字符之间',
                            trigger: 'blur'
                        }
                    ]
                }
            }
        },
        created() {
            this.getRolesList()
        },
        methods: {
            async getRolesList() {
                const { data: res } = await this.$http.get('roles')
                if(res.meta.status !== 200) {
                    return this.$message.error('获取角色列表失败')
                }
                this.rolelist = res.data
                console.log(this.rolelist)
            },
            async removeRightById(role, rightId) {
                const confirmResult = await this.$confirm(
                    '此操作将永久删除该权限，是否继续？',
                    '提示',
                    {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning'
                    }).catch(err => err)
                if(confirmResult !== 'confirm') {
                    return this.$message.info('已取消删除')
                }
                const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
                if(res.meta.status !== 200) {
                    return this.$message.error('删除权限失败')
                }
                role.children = res.data
            },
            async showSetRightDialog(role) {
                //展示分配权限的对话框
                //获取所有权限的数据
                this.roleId = role.id
                const { data: res } = await this.$http.get('rights/tree')
                if(res.meta.status !== 200) {
                    return this.$message.error('获取权限数据失败')
                }
                this.rightslist = res.data
                this.getLeafKeys(role, this.defKeys)
                this.setRightDialogVisible = true
            },
            getLeafKeys(node, arr) {
                //通过递归获取三级权限ID，并保存到defKeys中
                if(!node.children) {
                    return arr.push(node.id)
                }
                node.children.forEach(item => this.getLeafKeys(item, arr))
            },
            setRightDialogClosed() {
                this.defKeys = []
            },
            async allotRights() {
                const keys = [
                    ...this.$refs.treeRef.getCheckedKeys(),
                    ...this.$refs.treeRef.getHalfCheckedKeys()
                ]
                const idStr = keys.join(',')
                const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, {rids: idStr})
                if(res.meta.status !== 200) {
                    return this.$message.error('分配权限失败')
                }
                this.$message.success('分配权限成功')
                this.getRolesList()
                this.setRightDialogVisible = false
            },
            addDialogClosed() {
                this.$refs.addFormRef.resetFields()
            },
            addRole() {
                this.$refs.addFormRef.validate(async valid => {
                    if (!valid) return
                    const { data: res } = await this.$http.post('roles', this.addForm)
                    if(res.meta.status !== 201) {
                        return this.$message.error('添加角色失败')
                    }
                    this.$message.success('添加角色成功')
                    this.addDialogVisible = false
                    this.getRolesList()
                })
            },
            async showEditDialog(id) {
                const { data: res } = await this.$http.get('roles/' + id)
                if(res.meta.status !== 200) {
                    return this.$message.error('查询角色信息失败')
                }
                this.editForm = res.data
                this.editDialogVisible = true
            },
            editDialogClosed() {
                this.$refs.editFormRef.resetFields()
            },
            editRoleInfo() {
                this.$refs.editFormRef.validate(async valid => {
                    if (!valid) return
                    const { data: res } = await this.$http.put('roles/' + this.editForm.roleId, {roleName: this.editForm.roleName, roleDesc: this.editForm.roleDesc})
                    if(res.meta.status !== 200) {
                        return this.$message.error('更新角色信息失败')
                    }
                    this.$message.success('更新角色信息成功')
                    this.editDialogVisible = false
                    this.getRolesList()
                })
            },
            async removeRoleById(id) {
                const confirmResult = await this.$confirm(
                    '此操作将永久删除该角色，是否继续？',
                    '提示',
                    {
                        confirmButtonText: '确定',
                        cancelButtonText: '取消',
                        type: 'warning'
                    }
                ).catch(err => err)
                if (confirmResult !== 'confirm') {
                    return this.$message.info('已取消删除')
                }
                const { data: res } = await this.$http.delete('roles/' + id)
                if (res.meta.status !== 200) {
                    return this.$message.error('删除角色成功')
                }
                this.$message.success('删除角色成功')
                this.getRolesList()
            }
        }
    }
</script>

<style lang="less" scoped>
    .el-tag {
        margin: 7px;
    }
    .bdtop {
        border-top: 1px solid #eee;
    }
    .bdbottom {
        border-bottom: 1px solid #eee;
    }
    .vcenter {
        display: flex;
        align-items: center;
    }
</style>