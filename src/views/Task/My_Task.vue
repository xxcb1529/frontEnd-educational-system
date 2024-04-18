<template>
  <div>
    <el-tabs class="tab" v-model="activeName" @tab-click="tabClick">
      <el-tab-pane label="待办" name="待办">
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px">
          <el-form :inline="true" :model="filters" @submit.native.prevent>
            <el-form-item>
              <el-input v-model="filters.name" placeholder="任务名称"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="getTaskByName">查询</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-table :data="noDoneTableData" style="width: 100%; margin-bottom: 20px" border :row-class-name="tableRowClassName">
          <el-table-column type="selection" width="50"> </el-table-column>
          <el-table-column prop="name" label="任务名" width="150">
          </el-table-column>
          <el-table-column prop="priority" label="等级" width="80" sortable>
          </el-table-column>
          <el-table-column prop="start_time" label="开始时间" :formatter="formatStartTime" sortable>
          </el-table-column>
          <el-table-column prop="end_time" label="结束时间" :formatter="formatEndTime" sortable>
          </el-table-column>
          <el-table-column prop="description" label="详情描述">
          </el-table-column>
          <el-table-column prop="status" label="状态" :formatter="checkStatus" sortable>
            <template slot-scope="scope">
              <el-tag :type="getStatusTagType(scope.row.status)" disable-transitions>
                {{ checkStatus(scope.row) }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="create_by" label="创建人" width="80" sortable>
          </el-table-column>
          <el-table-column prop="CreateTime" label="创建时间" :formatter="formatCreateTime" sortable>
          </el-table-column>
          <el-table-column prop="remark" label="备注">
          </el-table-column>
          <el-table-column prop="ApprovalOpinions" label="审核意见">
          </el-table-column>
          <el-table-column label="操作" width="80" fixed="right">
            <template slot-scope="scope">
              <el-button type="primary" size="small" @click="handleApproval(scope.$index, scope.row)">提交</el-button>
            </template>
          </el-table-column>
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
          <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="50" :total="total"
            style="float: right">
          </el-pagination>
        </el-col>
      </el-tab-pane>
      <el-tab-pane label="已办" name="已办">
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0px">
          <el-form :inline="true" :model="filters" @submit.native.prevent>
            <el-form-item>
              <el-input v-model="filters.name" placeholder="任务名称"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="getTaskByName">查询</el-button>
            </el-form-item>
          </el-form>
        </el-col>
        <el-table :data="doneTableData" style="width: 100%; margin-bottom: 20px" border>
          <el-table-column type="selection" width="50"> </el-table-column>
          <el-table-column prop="name" label="任务名" width="150">
          </el-table-column>
          <el-table-column prop="priority" label="等级" width="80" sortable>
          </el-table-column>
          <el-table-column prop="start_time" label="开始时间" :formatter="formatStartTime" sortable>
          </el-table-column>
          <el-table-column prop="end_time" label="结束时间" :formatter="formatEndTime" sortable>
          </el-table-column>
          <el-table-column prop="description" label="详情描述">
          </el-table-column>
          <el-table-column prop="status" label="状态" :formatter="checkStatus" sortable>
            <template slot-scope="scope">
              <el-tag :type="getStatusTagType(scope.row.status)" disable-transitions>
                {{ checkStatus(scope.row) }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="create_by" label="创建人" width="80" sortable>
          </el-table-column>
          <el-table-column prop="CreateTime" label="创建时间" :formatter="formatCreateTime" sortable>
          </el-table-column>
          <el-table-column prop="remark" label="备注">
          </el-table-column>
          <el-table-column prop="ApprovalOpinions" label="审核意见">
          </el-table-column>
          <!-- <el-table-column label="操作" width="150" fixed="right">
        <template slot-scope="scope">
          <el-button size="small" @click="handleEdit(scope.$index, scope.row)">查看</el-button>
        </template>
      </el-table-column> -->
        </el-table>

        <!--工具条-->
        <el-col :span="24" class="toolbar">
          <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="50" :total="total"
            style="float: right">
          </el-pagination>
        </el-col>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
import util from "../../../util/date";
import {
  GetMyTasksByTeamIds,
  GetTasks,
  UpdateTask
} from "../../api/api";

export default {
  data() {
    return {
      doneTableData: [],
      tableData: [],
      noDoneTableData: [],
      options: [],
      filters: {
        name: "",
      },
      //   users: [],
      modules: [], //接口api列表
      statusList: [
        { Name: "激活", value: true },
        { Name: "禁用", value: false },
      ],
      total: 0,
      page: 1,
      listLoading: false,
      sels: [], //列表选中列
      activeName: "待办"
    };
  },
  methods: {
    tableRowClassName({row}){
      if(row.ApprovalOpinions !== null || row.ApprovalOpinions ===""){
        return 'warning-row';
      }
      return ''
    },
    getStatusTagType: function (status) {
      var statusTypeMap = {
        0: 'success', // 完成
        1: 'info',    // 进行中
        2: 'success', // 已完成
        3: 'danger',  // 取消
        4: 'warning', // 未开始
        5: 'danger',   // 未完成
        6: "info"
      };
      return statusTypeMap[status] || 'default';
    },
    checkStatus: function (row) {
      var statusDict = {
        0: "完成",
        1: "进行中",
        2: "已完成",
        3: "已取消",
        4: "未开始",
        5: "未完成"
      };

      return statusDict[row.status] || "未知";
    },
    formatStartTime: function (row, column) {
      return !row.start_time || row.start_time == ""
        ? ""
        : util.formatDate.format(new Date(row.start_time), "yyyy-MM-dd");
    },
    formatEndTime: function (row, column) {
      return !row.end_time || row.end_time == ""
        ? ""
        : util.formatDate.format(new Date(row.end_time), "yyyy-MM-dd");
    },
    formatCreateTime: function (row, column) {
      return !row.CreateTime || row.CreateTime == ""
        ? ""
        : util.formatDate.format(new Date(row.CreateTime), "yyyy-MM-dd");
    },
    handleCurrentChange(val) {
      this.page = val;
      this.getPersonTasks();
    },
    //获取个人任务一列表
    getPersonTasks() {
      let userInfo = JSON.parse(window.localStorage.getItem('user'))
      if (userInfo.RoleName !== 'Admin_Role') {
        let para = {
          ids: userInfo.team_ids
        };
        this.listLoading = true;
        GetMyTasksByTeamIds(para).then((res) => {
          const tasks = res.data.response;
          this.doneTableData = tasks.filter(task => task.done === true);
          this.noDoneTableData = tasks.filter(task => task.done === false);
          this.total = res.data.response.length;
          this.listLoading = false;
        }).catch(err => {
          console.log(err);
        })
      } else {
        let para = {
          page: this.page
        }
        GetTasks(para).then(res => {
          console.log(res.data.response.data);
          this.tableData = res.data.response.data;
          this.noDoneTableData = res.data.response.data
          this.total = res.data.response.dataCount;
        })
      }
    },
    getTaskByName: function () {
      let userInfo = JSON.parse(window.localStorage.getItem('user'))
      let para = {
        page: this.page,
        key: this.filters.name,
        teamids: userInfo.team_ids
      }
      GetTasks(para).then(res => {
        // console.log(res);
        this.tableData = res.data.response.data;
        this.total = res.data.response.dataCount;
      })
    },
    handleApproval(index, row) {
      this.$confirm('确认提交吗?', '提示', {
        type: 'warning'
      }).then(() => {
        let userInfo = JSON.parse(window.localStorage.getItem("user"))
        row.user_done_ids = []
        row.user_done_ids.push(userInfo.uID)
        row.user_done_ids = row.user_done_ids.join(",")
        UpdateTask(row).then(res => {
          if (res.data.success) {
            this.$message({
              message: "提交成功",
              type: "success",
            });
            setTimeout(() => {
              this.getPersonTasks()
            }, 500)
          } else {
            this.$message({
              message: res.data.msg,
              type: "error",
            });
          }
        })
      })

    },

    selsChange: function (sels) {
      this.sels = sels;
    },
    //批量修改状态
    batchRemove: function () {
      this.$message({
        message: "该功能未开放",
        type: "warning",
      });
    },

    tabClick(tab, event) {
      this.activeName = tab.name
    }
  },
  mounted() {
    this.getPersonTasks();
  },
};
</script>

<style scoped>
  .el-table .warning-row {
    background: oldlace;
  }

  .el-table .success-row {
    background: #f0f9eb;
  }</style>
