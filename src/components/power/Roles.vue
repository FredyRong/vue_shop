<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{path:'/home'}">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <el-row>
        <el-table :data="roleList" border stripe>
          <el-table-column type="expand">
            <template slot-scope="scope">
              <el-row :class="['bdbottom', i1===0?'bdtop': '', 'vcenter']" v-for="(item1, i1) in scope.row.children"
                      :key="item1.id">
                <el-col :span="5">
                  <el-tag closable
                          @close="removeRightById(scope.row, item1.id)">{{item1.authName}}
                  </el-tag>
                  <i class="el-icon-caret-right"></i>
                </el-col>
                <el-col :span="19">
                  <el-row :class="[i2 === 0? '':'bdtop', 'vcenter']" v-for="(item2, i2) in item1.children"
                          :key="item2.id">
                    <el-col :span="6">
                      <el-tag type="success" closable
                              @close="removeRightById(scope.row, item2.id)">{{item2.authName}}
                      </el-tag>
                      <i class="el-icon-caret-right"></i>
                    </el-col>
                    <el-col :span="18">
                      <el-tag type="warning" v-for="(item3, i3) in item2.children" :key="item3.id" closable
                              @close="removeRightById(scope.row, item3.id)">{{item3.authName}}
                      </el-tag>
                    </el-col>
                  </el-row>
                </el-col>
              </el-row>
            </template>
          </el-table-column>
          <el-table-column label="#" type="index"></el-table-column>
          <el-table-column label="角色名称" prop="roleName"></el-table-column>
          <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
          <el-table-column label="操作" width="300px">
            <template slot-scope="scope">
              <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑
              </el-button>
              <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
              <el-button size="mini" type="warning" icon="el-icon-setting" @click="showRightDialog(scope.row)">分配权限
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </el-row>
    </el-card>

    <el-dialog title="分配权限" :visible.sync="setRightDialogVisible" width="50%" @close="setRightDialogClosed">
      <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all
               :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="allotRights">确定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="添加角色" :visible.sync="addDialogVisible" width="50%" @close="setAddDialogClosed">
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

    <el-dialog title="修改角色" :visible.sync="editDialogVisible" width="50%" @close="setEditDialogClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取消</el-button>
        <el-button type="primary" @click="editRole">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        roleId: '',
        roleList: [],
        setRightDialogVisible: false,
        rightsList: [],
        treeProps: {
          label: 'authName',
          children: 'children'
        },
        defKeys: [],

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
            }
          ],
          roleDesc: [
            {
              required: true,
              message: '请输入角色描述',
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
            }
          ],
          roleDesc: [
            {
              required: true,
              message: '请输入角色描述',
              trigger: 'blur'
            }
          ]
        }
      }
    },
    created () {
      this.getRoleList()
    },
    methods: {
      async getRoleList () {
        const { data: res } = await this.$http.get('roles')
        if (res.meta.status != 200) {
          return this.$message.error('获取角色列表失败！')
        }
        this.roleList = res.data
      },
      async removeRightById (role, rightId) {
        const confirmResult = await this.$confirm(
          '此操作将永远删除该角色权限，是否继续？',
          '提示',
          {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }
        ).catch((err => err))
        if (confirmResult !== 'confirm') {
          return this.$message.info('取消删除！')
        }
        const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if (res.meta.status !== 200) {
          return this.$message.error('删除权限失败！')
        }
        // await this.getRoleList()
        role.children = res.data
      },
      async showRightDialog (role) {
        this.roleId = role.id
        const { data: res } = await this.$http.get('rights/tree')
        if (res.meta.status !== 200) {
          return this.$message.error('获取权限数据失败')
        }
        console.log()
        this.rightsList = res.data
        this.getDefKeys(role, this.defKeys)
        this.setRightDialogVisible = true
      },
      getDefKeys (node, arr) {
        if (!node.children) {
          return arr.push(node.id)
        }
        node.children.forEach(item => this.getDefKeys(item, arr))
      },
      setRightDialogClosed () {
        this.defKeys = []
      },
      allotRights: async function () {
        const keys = [
          ...this.$refs.treeRef.getCheckedKeys(),
          ...this.$refs.treeRef.getHalfCheckedKeys()
        ]
        const idStr = keys.join(',')
        const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
        if (res.meta.status !== 200) {
          return this.$message.error('分配权限失败！')
        }
        this.$message.success('分配权限成功！')
        await this.getRoleList()
        this.setRightDialogVisible = false
      },
      addRole () {
        this.$refs.addFormRef.validate(async valid => {
          if (!valid) return
          const { data: res } = await this.$http.post('roles', this.addForm)
          if (res.meta.status !== 201) {
            this.$message.error('添加角色失败！')
          }
          this.addDialogVisible = false
          this.$message.success('添加角色成功！')
          await this.getRoleList()
        })
      },
      setAddDialogClosed () {
        this.addForm = {}
      },
      async showEditDialog (id) {
        const { data: res } = await this.$http.get('roles/' + id)
        if (res.meta.status !== 200) {
          this.$message.error('查询角色信息失败')
        }
        this.editForm = res.data
        this.editDialogVisible = true
      },
      async editRole () {
        this.$refs.editFormRef.validate(async valid => {
          const {data: res} = await this.$http.put('roles/' + this.editForm.roleId, {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc
          })
          if(res.meta.status !== 200){
            return this.$message.error('修改角色信息失败！')
          }
          this.$message.success('修改角色信息成功！')
          this.editDialogVisible = false
          await this.getRoleList()
        })
      },
      setEditDialogClosed () {
        this.editForm = {}
      },
      async removeRoleById(id){
        const confirmResult = await this.$confirm('此操作将永久删除该角色，是否继续？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if(confirmResult !== 'confirm'){
          return this.$message.info('已取消删除')
        }
        const {data: res} = await this.$http.delete('roles/' + id)
        if(res.meta.status !== 200){
          return this.$message.error('删除角色失败！')
        }
        this.$message.success('删除用户成功！')
        await this.getRoleList()
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
