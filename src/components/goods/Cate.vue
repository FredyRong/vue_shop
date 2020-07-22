<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{path: '/home'}">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
        </el-col>
      </el-row>
      <el-row>
        <tree-table class="treeTable" :data="cateList" :columns="columns" :expand-type="false" :selection-type="false"
                    :show-row-hover="false" show-index index-text="#" border>
          <template slot="isok" slot-scope="scope">
            <i class="el-icon-success" v-if="scope.row.cat_deleted === false" style="color: lightgreen;"></i>
            <i class="el-icon-error" v-else style="color: #ff0000;"></i>
          </template>
          <template slot="order" slot-scope="scope">
            <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
            <el-tag type="warning" size="mini" v-else>三级</el-tag>
          </template>
          <template slot="opt" slot-scope="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditCateDialog(scope.row.cat_id)">编辑
            </el-button>
            <el-button size="mini" type="danger" icon="el-icon-edit" @click="removeCateById(scope.row.cat_id)">删除
            </el-button>
          </template>
        </tree-table>
        <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange"
                       :current-change="queryInfo.pagenum"
                       :page-sizes="[3,5,10,15]" :page-size="queryInfo.pagesize"
                       layout="total, sizes, prev, pager, next, jumper"
                       :total="total"></el-pagination>
      </el-row>
    </el-card>

    <el-dialog title="添加分类"
               :visible.sync="addCateDialogVisible"
               width="50%"
               @close="addCateFormClosed">
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px"
               class="demo-ruleForm">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级名称：">
          <el-cascader :options="parentCateList" :props="cascaderProps" v-model="selectedKeys"
                       @change="parentCateChanged" clearable></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="修改分类" :visible.sync="editCateDialogVisible" width="50%" @close="editCateFormClosed">
      <el-form :model="editCateForm" :rules="editCateFormRules" ref="editCateFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="editCateForm.cat_name"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editCate">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  export default {
    data () {
      return {
        queryInfo: {
          type: 3,
          pagenum: 1,
          pagesize: 5,
        },
        total: 0,
        cateList: [],
        columns: [
          {
            label: '分类名称',
            prop: 'cat_name'
          },
          {
            label: '是否有效',
            type: 'template',
            template: 'isok'
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
          value: 'cat_id',
          label: 'cat_name',
          children: 'children',
          expandTrigger: 'hover',
          checkStrictly: true
        },
        selectedKeys: [],

        editCateDialogVisible: false,
        editCateForm: {},
        editCateFormRules: {
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
    created () {
      this.getCateList()
    },
    methods: {
      async getCateList () {
        const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品分类失败！')
        }
        this.cateList = res.data.result
        this.total = res.data.total
      },
      handleSizeChange (newSize) {
        this.queryInfo.pagesize = newSize
        this.getCateList()
      },
      handleCurrentChange (newPage) {
        this.queryInfo.pagenum = newPage
        this.getCateList()
      },
      showAddCateDialog () {
        this.getParentCateList()
        this.addCateDialogVisible = true
      },
      async getParentCateList () {
        const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
        if (res.meta.status !== 200) {
          return this.$message.error('获取父级分类数据失败！')
        }
        this.parentCateList = res.data
      },
      parentCateChanged () {
        if (this.selectedKeys.length > 0) {
          this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1]
          this.addCateForm.cat_level = this.selectedKeys.length
          return
        } else {
          this.addCateForm.cat_pid = 0
          this.addCateForm.cat_level = 0
        }
      },
      addCate () {
        this.$refs.addCateFormRef.validate(async valid => {
          if (!valid) return
          const { data: res } = await this.$http.post('categories', this.addCateForm)
          if (res.meta.status !== 201) {
            return this.$message.error('添加分类失败！')
          }
          this.$message.success('添加分类成功！')
          await this.getCateList()
          this.addCateDialogVisible = false
        })
      },
      addCateFormClosed () {
        this.$refs.addCateFormRef.resetFields()
        this.selectedKeys = []
        this.addCateForm.cat_level = 0
        this.addCateForm.cat_pid = 0
      },
      async showEditCateDialog (id) {
        const { data: res } = await this.$http.get('categories/' + id)
        if (res.meta.status !== 200) {
          return this.$message.error('查询分类失败！')
        }
        this.editCateForm = res.data
        this.editCateDialogVisible = true
      },
      async editCate () {
        const { data: res } = await this.$http.put('categories/' + this.editCateForm.cat_id, { cat_name: this.editCateForm.cat_name })
        if (res.meta.status !== 200) {
          return this.$message.error('修改分类名称失败！')
        }
        this.$message.success('修改分类名称成功！')
        this.editCateDialogVisible = false
        await this.getCateList()
      },
      editCateFormClosed () {
        this.$refs.editCateFormRef.resetFields()
        this.editCateForm = {}
      },
      async removeCateById (id) {
        const confirmResult = await this.$confirm('此操作将永久删除该类别，是否继续？', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        if (confirmResult !== 'confirm') {
          return this.$message.info('已取消删除')
        }
        const { data: res } = await this.$http.delete('categories/' + id)
        if(res.meta.status !== 200){
          return this.$message.error('删除用户失败！')
        }
        this.$message.success('删除用户成功！')
        await this.getCateList()
      }
    }
  }
</script>

<style lang="less" scoped>
  .treeTable {
    margin-top: 15px;
  }


</style>
