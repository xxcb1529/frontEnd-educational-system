<template>
    <div>
        <el-tabs class="tab" v-model="activeName" @tab-click="tabClick">
            <el-tab-pane label="待审批" name="待审批">
                <!--工具条-->
                <el-col :span="24" class="toolbar" style="padding-bottom: 0px">
                    <el-form :inline="true" :model="filters" @submit.native.prevent>
                        <el-form-item>
                            <el-input v-model="filters.name" placeholder="任务名称"></el-input>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="getApprovalTasks">查询</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>
                <el-table :data="approveTableData" style="width: 100%; margin-bottom: 20px" border>
                    <el-table-column type="selection" width="50"> </el-table-column>
                    <el-table-column prop="name" label="任务名" width="150">
                    </el-table-column>
                    <el-table-column prop="Author" label="提交人" width="80" sortable>
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
                    <el-table-column label="操作" width="80" fixed="right">
                        <template slot-scope="scope">
                            <el-button type="primary" size="small"
                                @click="handleApproval(true, scope.row)">审批</el-button>
                        </template>
                    </el-table-column>
                </el-table>

                <!--工具条-->
                <el-col :span="24" class="toolbar">
                    <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="50"
                        :total="total" style="float: right">
                    </el-pagination>
                </el-col>
            </el-tab-pane>
            <el-tab-pane label="已审批" name="已审批">
                <!--工具条-->
                <el-col :span="24" class="toolbar" style="padding-bottom: 0px">
                    <el-form :inline="true" :model="filters" @submit.native.prevent>
                        <el-form-item>
                            <el-input v-model="filters.name" placeholder="任务名称"></el-input>
                        </el-form-item>
                        <el-form-item>
                            <el-button type="primary" @click="getApprovalTasks">查询</el-button>
                        </el-form-item>
                    </el-form>
                </el-col>
                <el-table :data="approvedTableData" style="width: 100%; margin-bottom: 20px" border>
                    <el-table-column type="selection" width="50"> </el-table-column>
                    <el-table-column prop="name" label="任务名" width="150">
                    </el-table-column>
                    <el-table-column prop="Author" label="提交人" width="80" sortable>
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
                    <el-table-column prop="ApprovalOpinions" label="审批意见">
                    </el-table-column>
                    <el-table-column label="操作" width="80" fixed="right">
                        <template slot-scope="scope">
                            <el-button type="primary" size="small"
                                @click="handleApproval(false, scope.row)">驳回</el-button>
                        </template>
                    </el-table-column>
                </el-table>

                <!--工具条-->
                <el-col :span="24" class="toolbar">
                    <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="50"
                        :total="total" style="float: right">
                    </el-pagination>
                </el-col>
            </el-tab-pane>
        </el-tabs>
        <el-dialog v-if="dialogVisible" width="30%" :modal-append-to-body="false" title="审批"
            :visible.sync="dialogVisible" center>
            <el-input type="textarea" :rows="4" placeholder="请输入审批意见" v-model="approvalOption">
            </el-input>
            <span slot="footer" class="dialog-footer">
                <el-button @click="submitOption(false)">驳 回</el-button>
                <el-button v-if="showPass" type="primary" @click="submitOption(true)">通 过</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
import util from "../../../util/date";
import {
    GetApprovalTasks,
    ApprovalTask
} from "../../api/api";

export default {
    data() {
        return {
            tableData: [],
            approveTableData: [],
            approvedTableData: [],
            options: [],
            filters: {
                name: "",
            },
            statusList: [
                { Name: "激活", value: true },
                { Name: "禁用", value: false },
            ],
            total: 0,
            page: 1,
            listLoading: false,
            sels: [], //列表选中列
            activeName: "待审批",
            dialogVisible: false,
            approvalOption: "",
            TaskRow: {},
            showPass: true
        };
    },
    methods: {
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
            this.getApprovalTasks();
        },
        //获取待审批任务列表
        getApprovalTasks() {
            let userInfo = JSON.parse(window.localStorage.getItem('user'))
            let para = {
                userId: userInfo.uID,
                page: this.page,
                key: this.filters.name
            };
            this.listLoading = true;
            GetApprovalTasks(para).then((res) => {
                this.tableData = res.data.response;
                this.approveTableData = res.data.response.filter(task => task.IsApproved === false);
                this.approvedTableData = res.data.response.filter(task => task.IsApproved === true);
                this.total = res.data.response.length;
                this.listLoading = false;
            }).catch(err => {
                console.log(err);
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
        },
        // 打开对话框
        handleApproval(show, row) {
            this.TaskRow = row
            this.showPass = show
            this.dialogVisible = true
        },
        submitOption(ispass) {
            if (this.approvalOption === "") {
                this.$message({
                    message: "请填写审批意见",
                    type: "warning",
                });
                return
            }
            let para = {
                taskId: this.TaskRow.Id,
                userId: JSON.parse(window.localStorage.getItem("user")).uID,
                approvalOption: this.approvalOption,
                Ispass: ispass
            }
            ApprovalTask(para).then(res => {
                if (res.data.success) {
                    this.dialogVisible = false
                    this.approvalOption = ""
                    this.$message({
                        message: res.data.msg,
                        type: "success",
                    });
                    setTimeout(() => {
                        this.getApprovalTasks()
                    }, 500)
                } else {
                    this.$message({
                        message: res.data.msg,
                        type: "error",
                    });
                }
            }).catch(err => {
                console.log(err);
            })
        }
    },
    mounted() {
        this.getApprovalTasks();
    },
};
</script>

<style scoped></style>