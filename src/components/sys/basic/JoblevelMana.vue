<template>
  <div>
    <el-input
        size="small"
        v-model="jl.name"
        placeholder="添加职称等级..."
        prefix-icon="el-icon-plus"
        style="width: 300px"></el-input>
    <el-select
        size="small"
        v-model="jl.titleLevel"
        placeholder="职称等级"
        style="margin-left: 6px;margin-right: 6px">
      <el-option
          v-for="item in titleLevels"
          :key="item"
          :label="item"
          :value="item">
      </el-option>
    </el-select>
    <el-button size="small" type="primary" icon="el-icon-plus" @click="addJobLevel">添加</el-button>
    <el-table
        :data="jls"
        stripe
        border
        size="small"
        style="width: 70%;margin-top: 10px"
        @selection-change="handleSelectionChange">
      <el-table-column
          type="selection"
          width="55">
      </el-table-column>
      <el-table-column
          prop="id"
          label="编号"
          width="55">
      </el-table-column>
      <el-table-column
          prop="name"
          label="职称名"
          width="150">
      </el-table-column>
      <el-table-column
          prop="titleLevel"
          label="职称等级"
          width="150">
      </el-table-column>
      <el-table-column
          prop="createDate"
          label="创建日期"
          width="150">
      </el-table-column>
      <el-table-column
          prop="enabled"
          label="是否启用"
          width="150">
        <!--这里布尔值无法直接显示，需要使用具名插槽，自定义模板-->
        <template slot-scope="scope">
          <el-tag v-if="scope.row.enabled" type="success">已启用</el-tag>
          <el-tag v-else type="danger">未启用</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <!--使用具名插槽，自定义模板-->
        <template slot-scope="scope">
          <el-button
              size="small"
              @click="showEditView(scope.row)">编辑
          </el-button>
          <el-button
              size="small"
              type="danger"
              @click="deleteHandler(scope.row)">删除
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-dialog
        size="small"
        title="编辑职称"
        :visible.sync="dialogVisible"
        width="30%">
      <table>
        <tr>
          <td>
            <el-tag>职称名称</el-tag>
          </td>
          <td>
            <el-input size="small" v-model="updateJl.name" style="margin-left: 6px"></el-input>
          </td>
        </tr>
        <tr>
          <td>
            <el-tag>职称等级</el-tag>
          </td>
          <td>
            <el-select
                size="small"
                v-model="updateJl.titleLevel"
                placeholder="职称等级"
                style="margin-left: 6px;margin-right: 6px">
              <el-option
                  v-for="item in titleLevels"
                  :key="item"
                  :label="item"
                  :value="item">
              </el-option>
            </el-select>
          </td>
        </tr>
        <tr>
          <td>
            <el-tag>是否启用</el-tag>
          </td>
          <td>
            <el-switch
                style="margin-left: 6px"
                v-model="updateJl.enabled"
                active-color="#13ce66"
                inactive-color="#ff4949"
                active-text="已启用"
                inactive-text="未启用">
            </el-switch>
          </td>
        </tr>
      </table>
      <span slot="footer" class="dialog-footer">
        <el-button size="small" @click="dialogVisible = false">取 消</el-button>
        <el-button size="small" type="primary" @click="doUpdate">确 定</el-button>
      </span>
    </el-dialog>
    <el-button
        type="danger"
        size="small"
        style="margin-top: 10px"
        :disabled="!this.multipleSelection.length"
        @click="deleteMany">
      批量删除
    </el-button>
  </div>
</template>

<script>
export default {
  name: "JoblebelMana",
  data() {
    return {
      jl: {
        name: '',
        titleLevel: ''
      },
      updateJl: {
        name: '',
        titleLevel: '',
        enabled: true
      },
      titleLevels: [
        '正高级',
        '副高级',
        '中级',
        '初级',
        '员级'
      ],
      jls: [],
      dialogVisible: false,
      multipleSelection: []
    }
  },
  methods: {
    initJls() {
      this.getRequest('/system/basic/joblevel/').then(resp => {
        if (resp) {
          this.jls = resp
          this.jl.name = ''
          this.jl.titleLevel = ''
        }
      })
    },
    addJobLevel() {
      if (this.jl.name && this.jl.titleLevel) {
        this.postRequest('/system/basic/joblevel/', this.jl).then(resp => {
          if (resp) {
            this.initJls();
          }
        });
      } else {
        this.$message.error("字段不能为空")
      }
    },
    deleteHandler(data) {
      this.$confirm(`此操作将永久删除${data.name}职称, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.deleteRequest(`/system/basic/joblevel/${data.id}`).then(resp => {
          if (resp) {
            this.initJls()
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
    showEditView(data) {
      Object.assign(this.updateJl, data)
      this.dialogVisible = true;
      this.updateJl.createDate = ''
    },
    doUpdate() {
      this.putRequest('/system/basic/joblevel/', this.updateJl).then(resp => {
        if (resp) {
          this.initJls()
          this.dialogVisible = false
        }
      })
    },
    handleSelectionChange(val) {
      this.multipleSelection = val
    },
    deleteMany() {
      this.$confirm(`此操作将永久删除这${this.multipleSelection.length}条职称, 是否继续?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        let ids = '?'
        this.multipleSelection.forEach(item => {
          ids += `ids=${item.id}&`
        })
        this.deleteRequest(`/system/basic/joblevel/${ids}`).then(resp => {
          if (resp) {
            this.initJls()
          }
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        });
      });
    },
  },
  mounted() {
    this.initJls()
  }
}
</script>

<style scoped>

</style>