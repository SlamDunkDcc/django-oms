<template>
  <div class="components-container" style='height:100vh'>
    <el-card>
      <div class="head-lavel">
        <div class="table-button">
          <el-button type="primary" icon="el-icon-plus" @click="addForm=true">新建</el-button>
          <el-select v-model="listQuery.status" placeholder="请选择状态筛选" clearable @change="changeTeststatus">
            <el-option
              v-for="item in Object.keys(Test_Status)"
              :key="item"
              :label="Test_Status[item]"
              :value="item">
            </el-option>
          </el-select>
        </div>
        <div class="table-search">
          <el-input
            placeholder="搜索..."
            v-model="listQuery.search"
            @keyup.enter.native="searchClick">
            <i class="el-icon-search el-input__icon" slot="suffix" @click="searchClick"></i>
          </el-input>
        </div>
      </div>
      <div>
        <el-table :data='tableData' border style="width: 100%">
          <el-table-column type="expand">
            <template slot-scope="props">
              <el-form label-position="left" inline class="table-expand">
                <el-form-item label="测试人员" prop="test_user">
                  <span>{{ props.row.test_user }}</span>
                </el-form-item>
                <el-form-item label="开发人员" prop="action_user">
                  <span>{{ props.row.action_user }}</span>
                </el-form-item>
                <el-form-item label="测试时间" prop="test_time">
                  <span>{{ props.row.test_time }}</span>
                </el-form-item>
              </el-form>
            </template>
          </el-table-column>
          <el-table-column prop='id' label='编号'></el-table-column>
          <el-table-column prop='name' label='名称'></el-table-column>
          <el-table-column prop='expect_result' label='预期结果'></el-table-column>
          <el-table-column prop='actual_result' label='实际结果'></el-table-column>
          <el-table-column prop='status' label='执行状态'>
            <template slot-scope="scope">
              <div slot="reference">
                <el-tag size="mini">{{Test_Status[scope.row.status]}}</el-tag>
              </div>
            </template>
          </el-table-column>
          <el-table-column prop='project' label='关联任务'>
            <template slot-scope="scope">
              <div slot="reference">
                <el-button type="text" @click="showProject(scope.row.project)">{{scope.row.project}}</el-button>
              </div>
            </template>
          </el-table-column>
          <el-table-column prop='create_time' label='创建时间'>
            <template slot-scope="scope">
              <div slot="reference" style="text-align: center; color: rgb(0,0,0)">
                <span>{{scope.row.create_time | parseDate}}</span>
              </div>
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <el-button @click="handleEdit(scope.row)" type="success" size="small">修改</el-button>
              <el-button @click="deleteGroup(scope.row.id)" type="danger" size="small">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="table-pagination">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page.sync="currentPage"
          :page-sizes="pagesize"
          :page-size="listQuery.limit"
          :layout="pageformat"
          :total="tabletotal">
        </el-pagination>
      </div>
    </el-card>
    <el-dialog :visible.sync="addForm">
      <add-group @DialogStatus="getDialogStatus"></add-group>
    </el-dialog>
    <el-dialog :visible.sync="editForm" @close="closeEditForm">
      <edit-group :ruleForm="rowdata" @DialogStatus="getDialogStatus"></edit-group>
    </el-dialog>
    <el-dialog :visible.sync="showprojectForm">
      <show-project :ruleForm="project"></show-project>
    </el-dialog>
  </div>
</template>

<script>
import { getTestManager, deleteTestManager, getProject } from '@/api/project'
import { LIMIT, pagesize, pageformat } from '@/config'
import addGroup from './components/addtest.vue'
import editGroup from './components/edittest.vue'
import showProject from './components/showproject.vue'

export default {
  components: {
    addGroup, editGroup, showProject
  },
  data() {
    return {
      tableData: [],
      tabletotal: 0,
      currentPage: 1,
      pagesize: pagesize,
      pageformat: pageformat,
      listQuery: {
        limit: LIMIT,
        offset: '',
        status: '',
        search: ''
      },
      addForm: false,
      editForm: false,
      rowdata: {},
      Test_Status: {
        0: 'Passed',
        1: 'Failed',
        2: 'Block',
        3: 'N/A',
        4: 'New'
      },
      project: '',
      showprojectForm: false
    }
  },

  created() {
    this.fetchData()
  },

  methods: {
    fetchData() {
      getTestManager(this.listQuery).then(response => {
        this.tableData = response.data.results
        this.tabletotal = response.data.count
      })
    },
    getDialogStatus(data) {
      this.addForm = data
      this.editForm = data
      setTimeout(this.fetchData, 1000)
    },
    deleteGroup(id) {
      deleteTestManager(id).then(response => {
        this.$message({
          message: '恭喜你，删除成功',
          type: 'success'
        })
        this.fetchData()
      }).catch(error => {
        this.$message.error('删除失败')
        console.log(error)
      })
    },
    closeEditForm() {
      this.fetchData()
    },
    handleEdit(row) {
      this.editForm = true
      this.rowdata = row
    },
    searchClick() {
      this.fetchData()
    },
    handleSizeChange(val) {
      this.listQuery.limit = val
      this.fetchData()
    },
    handleCurrentChange(val) {
      this.listQuery.offset = (val - 1) * LIMIT
      this.fetchData()
    },
    showProject(pid) {
      this.showprojectForm = true
      const query = {
        pid: pid
      }
      getProject(query).then(response => {
        this.project = response.data[0]
      })
    },
    changeTeststatus() {
      this.fetchData()
    }
  }
}
</script>

<style lang='scss'>

</style>
