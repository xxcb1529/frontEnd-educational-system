<template>
    <div class="body">
        <el-row v-if="taskData">
            <el-col :span="8" v-for="(item, index) in taskData" :key="item.Id">
                <div class="task-content">
                    <div class="team-name">
                        {{ item.team_name }}
                        <div class="info" @click="handleInfo(item.Id)">详细</div>
                    </div>
                    <div>
                        <div class="children" v-for="(task, index) in item.children" :key="task.Id">
                            <div class="left">
                                <div style="width: 6px;height: 16px;margin-right: 0.5rem;"
                                    :style="{ backgroundColor: getColor(task.status) }"></div>{{ task.name }}
                            </div>
                            <div class="status" :style="{ color: getColor(task.status) }">
                                <i class="fa" :style="{ color: getColor(task.status) }"
                                    :class="getStatusTagType(task.status)"></i>
                                {{ checkStatus(task.status) }}
                            </div>
                        </div>
                    </div>
                </div>
            </el-col>
            <!--详细界面-->
            <el-dialog title="详细" width="80%" :visible.sync="TaskTableVisible" v-model="TaskTableVisible"
                :close-on-click-modal="false">
                <!--工具条-->
                <el-col :span="24" class="toolbar" style="padding-bottom: 0px">
                    <el-form :inline="true" :model="filters" @submit.native.prevent>
                        <el-form-item>
                            <el-input v-model="filters.Name" placeholder="任务名称"></el-input>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="getTaskByName">查询</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>
                <el-table :data="tableData" style="width: 100%; margin-bottom: 20px" border>
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
                            <el-tag :type="getStatusTypeTag(scope.row.status)" disable-transitions>
                                {{ checkStatus(scope.row.status) }}
                            </el-tag>
                        </template>
                    </el-table-column>
                    <el-table-column prop="create_by" label="创建人" width="80" sortable>
                    </el-table-column>
                    <el-table-column prop="CreateTime" label="创建时间" :formatter="formatCreateTime" sortable>
                    </el-table-column>
                    <el-table-column prop="remark" label="备注">
                    </el-table-column>
                    <!-- <el-table-column label="操作" width="150" fixed="right">
        <template slot-scope="scope">
          <el-button size="small" @click="handleEdit(scope.$index, scope.row)">查看</el-button>
        </template>
      </el-table-column> -->
                </el-table>
                <!--工具条-->
                <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="50"
                    :total="total" style="float: right">
                </el-pagination>
            </el-dialog>
        </el-row>
        <div v-else class="nothing">
            <img src="../../assets/暂无公告.png" alt="">
            No Team Participation
        </div>
    </div>
</template>
<script>
import util from "../../../util/date";

import {
    GetTasksByTeamIds,
    DelTask,
    getTeamTasksByTeamId,
    GetTasks
} from "../../api/api";
export default {
    data() {
        return {
            taskData: [],
            TaskTableVisible: false,
            tableData: [],
            getTeamTasksInfoLoading: false,
            filters: {
                Name: "",
            },
            page: 1,
            taskInfoId: 0,
            total: 0
        }
    },
    methods: {
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
        getTeamTask() {
            let userInfo = JSON.parse(window.localStorage.getItem('user'))
            let para = {
                ids: userInfo.team_ids
            };
            GetTasksByTeamIds(para).then((res) => {
                this.taskData = res.data.response
            }).catch(err => {
                console.log(err);
            })
        },
        handleCurrentChange(val) {
            this.page = val;
            this.getTaskByName();
        },
        getColor: function (status) {
            var statusTypeMap = {
                0: '#A3D6F1', // 完成
                1: 'yellow',    // 进行中
                2: '#A3D6F1', // 已完成
                3: '#EF95C1',  // 取消
                4: '#FBE2EE', // 未开始
                5: 'red',   // 未完成
            };
            return statusTypeMap[status] || 'default';
        },
        getStatusTypeTag: function (status) {
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
        getStatusTagType: function (status) {
            var statusTypeMap = {
                0: 'fa-check-circle-o', // 完成
                1: 'fa-clock-o',    // 进行中
                2: 'fa-check-circle-o', // 已完成
                3: 'fa-times-circle-o',  // 取消
                4: 'fa-minus-circle', // 未开始
                5: 'fa-exclamation-circle',   // 未完成
            };
            return statusTypeMap[status] || 'default';
        },
        checkStatus: function (status) {
            var statusDict = {
                0: "完成",
                1: "进行中",
                2: "已完成",
                3: "已取消",
                4: "未开始",
                5: "未完成"
            };

            return statusDict[status] || "未知";
        },
        //删除
        handleDel: function (index, row) {
            this.$confirm("确认删除吗?", "提示", {
                type: "warning",
            })
                .then(() => {
                    this.listLoading = true;
                    //NProgress.start();
                    let para = { id: row.Id };
                    DelTask(para).then((res) => {
                        if (util.isEmt.format(res)) {
                            this.listLoading = false;
                            return;
                        }
                        this.listLoading = false;
                        //NProgress.done();
                        if (res.data.success) {
                            this.$message({
                                message: "删除成功",
                                type: "success",
                            });
                        } else {
                            this.$message({
                                message: res.data.msg,
                                type: "error",
                            });
                        }

                        this.getTeamTask();
                    });
                })
                .catch(() => { });
        },
        //显示任务详情界面
        handleInfo: function (team_id) {
            let that = this;
            this.taskInfoId = team_id
            that.getTeamTasksInfoLoading = true;

            this.TaskTableVisible = true;

            this.options = [];
            let para = { id: team_id };
            getTeamTasksByTeamId(para).then((res) => {
                this.tableData = res.data.response
                that.getTeamTasksInfoLoading = false;
            });
        },
        getTaskByName: function () {
            let para = {
                page: this.page,
                key: this.filters.Name,
                teamids: this.taskInfoId.toString()
            }

            GetTasks(para).then(res => {
                this.tableData = res.data.response.data;
                this.total = res.data.response.dataCount;
            })
        },
    },
    mounted() {
        this.getTeamTask()
    }
}
</script>
<style scoped lang="less">
.nothing {
    position: fixed;
    bottom: 50%;
    right: 50%;
    transform: translate(50%, 50%);
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 200px;
    height: 220px;
    font-size: 20px;
    font-weight: 600;
    font-family: 'Times New Roman', Times, serif;
    color: rgba(0, 0, 0, .2);

    img {
        width: 100%;
        height: auto;
    }
}

.el-row {
    margin-bottom: 20px;
    overflow: hidden;
    height: 100%;
    padding: 20px;

    &:last-child {
        margin-bottom: 0;
    }
}

.el-col {
    border-radius: 4px;
    padding: 20px;
    // background: #f7f8f9;
    box-shadow: 0px 0px 15px 0px rgba(233, 237, 248, 0.70);
}

.bg-purple-dark {
    background: #99a9bf;
}

.bg-purple-light {
    background: #e5e9f2;
}

.task-content {
    border-radius: 4px;
    display: flex;
    flex-direction: column;
    padding: 0 20px;

    .team-name {
        font-size: 20px;
        font-weight: 600;
        margin-bottom: 20px;
        display: flex;
        align-items: center;
        justify-content: space-between;

        .info {
            cursor: pointer;
            font-weight: lighter;
            font-size: 16px;

            &:hover {
                color: rgb(255, 208, 75);
            }
        }
    }

    .children {
        display: flex;
        align-items: center;
        justify-content: space-between;
        margin-bottom: 5px;

        .left {
            display: flex;
            align-items: center;
            justify-content: space-between;
        }
    }
}
</style>