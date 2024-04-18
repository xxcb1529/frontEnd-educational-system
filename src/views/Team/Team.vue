<template>
    <div>
        <section>
            <!--工具条-->
            <el-col :span="24" class="toolbar" style="padding-bottom: 0px">
                <el-form :inline="true" :model="filters" @submit.native.prevent>
                    <el-form-item>
                        <el-input v-model="filters.name" placeholder="团队名称"></el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="getTeam">查询</el-button>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="addTeam">新增</el-button>
                    </el-form-item>
                </el-form>
            </el-col>
            <el-table :data="tableData" style="width: 100%; margin-bottom: 20px" border>
                <el-table-column type="selection" width="50"> </el-table-column>
                <el-table-column prop="name" label="团队" width="150">
                </el-table-column>
                <el-table-column prop="type" label="类型" width="80" sortable>
                    <template slot-scope="scope">
                        {{ getTeamType(scope.row.type) }}
                    </template>
                </el-table-column>
                <el-table-column label="负责人" width="80">
                    <template scope="scope">
                        <el-button size="small" @click="lookHeaderInfo(scope.row.Id)">查看</el-button>
                    </template>
                </el-table-column>
                <el-table-column label="成员" width="80">
                    <template scope="scope">
                        <el-button size="small" @click="lookMembersInfo(scope.row.member_ids)">查看</el-button>
                    </template>
                </el-table-column>
                <el-table-column label="任务" width="80">
                    <template scope="scope">
                        <el-button size="small" @click="lookTasksInfo(scope.row.Id)">查看</el-button>
                    </template>
                </el-table-column>
                <el-table-column label="管理团队" width="80">
                    <template scope="scope">
                        <el-button size="small" @click="lookTeamsInfo(scope.row.subordinate_team_ids)">查看</el-button>
                    </template>
                </el-table-column>
                <!-- <el-table-column prop="head_id" label="负责人">
            </el-table-column> -->
                <!-- <el-table-column prop="member_ids" label="成员">
            </el-table-column> -->
                <!-- <el-table-column prop="task_ids" label="任务">
            </el-table-column> -->
                <!-- <el-table-column prop="subordinate_team_ids" label="管理团队">
            </el-table-column> -->
                <el-table-column prop="belongs" label="隶属">
                </el-table-column>
                <el-table-column prop="contact" label="联系方式">
                </el-table-column>
                <el-table-column prop="status" label="状态" :formatter="checkTeamStatus" sortable>
                    <template slot-scope="scope">
                        <el-tag :type="getTeamStatusTagType(scope.row.status)" disable-transitions>
                            {{ checkTeamStatus(scope.row) }}
                        </el-tag>
                    </template>
                </el-table-column>
                <el-table-column prop="create_by" label="创建人" width="80" sortable>
                </el-table-column>
                <el-table-column prop="CreateTime" label="创建时间" :formatter="formatCreateTime" sortable>
                </el-table-column>
                <el-table-column prop="remark" label="备注">
                </el-table-column>
                <el-table-column label="操作" width="150">
                    <template scope="scope">
                        <el-button size="small" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
                        <el-button type="danger" size="small" @click="handleDel(scope.row.Id)">{{
                scope.row.status === 0 ? "解散" : "删除" }} </el-button>
                    </template>
                </el-table-column>
            </el-table>
            <!--工具条-->
            <el-col :span="24" class="toolbar">
                <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="50"
                    :total="total" style="float: right">
                </el-pagination>
            </el-col>
        </section>
        <!-- 新增 -->
        <section>
            <el-dialog title="新增" :visible.sync="addFormVisible" v-model="addFormVisible" :close-on-click-modal="false">
                <el-form :model="addForm" label-width="80px" :rules="addFormRules" ref="addForm">
                    <el-form-item label="团队名称" prop="name">
                        <el-input v-model="addForm.name" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="团队类型" prop="type">
                        <el-select v-model="addForm.type" placeholder="请选择团队类型">
                            <el-option v-for="item in teamType" :key="item.value" :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="团队成员" prop="member_ids">
                        <el-select v-model="addForm.member_ids" filterable multiple placeholder="请添加团队成员"
                            @change="handleSelectChange">
                            <el-option v-for="item in allTeachers" :key="item.value" :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="负责人" prop="head_id">
                        <el-select v-model="addForm.head_id" placeholder="请选择负责人">
                            <el-option v-for="item in selectTeachers" :key="item.value" :label="item.label"
                                :value="item.value"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="管理团队" prop="subordinate_team_ids">
                        <el-select v-model="addForm.subordinate_team_ids" filterable multiple placeholder="请选择团队">
                            <el-option v-for="item in allTeams" :key="item.value" :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item prop="belongs" label="隶属于" width="" sortable>
                        <el-input v-model="addForm.belongs" auto-complete="off"> </el-input>
                    </el-form-item>
                    <el-form-item prop="contact" label="联系方式" width="" sortable>
                        <el-input v-model="addForm.contact" auto-complete="off"> </el-input>
                    </el-form-item>
                    <el-form-item prop="remark" label="备注" width="" sortable>
                        <el-input v-model="addForm.remark" auto-complete="off"> </el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click.native="addFormVisible = false">取消</el-button>
                    <el-button type="primary" @click.native="addSubmit" :loading="addLoading">提交</el-button>
                </div>
            </el-dialog>
        </section>
        <!-- 编辑 -->
        <section>
            <el-dialog title="编辑" :visible.sync="editFormVisible" v-model="editFormVisible"
                :close-on-click-modal="false">

                <el-form :model="editForm" label-width="80px" :rules="editFormRules" ref="editForm">
                    <el-form-item label="团队名称" prop="name">
                        <el-input v-model="editForm.name" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="团队类型" prop="type">
                        <el-select v-model="editForm.type" placeholder="请选择团队类型">
                            <el-option v-for="item in teamType" :key="item.value" :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="团队成员" prop="member_ids">
                        <el-select v-model="editForm.member_ids" filterable multiple placeholder="请添加团队成员"
                            @change="handleSelectChange">
                            <el-option v-for="item in allTeachers" :key="item.value" :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="负责人" prop="head_id">
                        <el-select v-model="editForm.head_id" placeholder="请选择负责人">
                            <el-option v-for="item in selectTeachers" :key="item.value" :label="item.label"
                                :value="item.value"></el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="团队任务" prop="task_ids">
                        <el-select v-model="editForm.task_ids" filterable multiple placeholder="请选择团队">
                            <el-option v-for="item in selectTeamTask" :key="item.value" :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="管理团队" prop="subordinate_team_ids">
                        <el-select v-model="editForm.subordinate_team_ids" filterable multiple placeholder="请选择团队">
                            <el-option v-for="item in allTeams" :key="item.value" :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item prop="belongs" label="隶属于" width="" sortable>
                        <el-input v-model="editForm.belongs" auto-complete="off"> </el-input>
                    </el-form-item>
                    <el-form-item prop="contact" label="联系方式" width="" sortable>
                        <el-input v-model="editForm.contact" auto-complete="off"> </el-input>
                    </el-form-item>
                    <el-form-item prop="remark" label="备注" width="" sortable>
                        <el-input v-model="editForm.remark" auto-complete="off"> </el-input>
                    </el-form-item>
                </el-form>
                <div slot="footer" class="dialog-footer">
                    <el-button @click.native="editFormVisible = false">取消</el-button>
                    <el-button type="primary" @click.native="editSubmit" :loading="editLoading">提交</el-button>
                </div>
            </el-dialog>
        </section>
        <!-- 查看团队成员 -->
        <section>
            <el-dialog title="成员信息" width="80%" :visible.sync="TeamTableVisible" v-model="TeamTableVisible"
                :close-on-click-modal="false">
                <el-table :data="TeamMembersableData" style="width: 100%; margin-bottom: 20px" border>
                    <el-table-column type="selection" width="50"></el-table-column>
                    <el-table-column type="index" width="80"></el-table-column>
                    <el-table-column prop="TeacherNo" label="工号" width sortable></el-table-column>
                    <el-table-column prop="Name" label="教师" width sortable></el-table-column>
                    <el-table-column prop="iNumber" label="联系电话"></el-table-column>
                    <el-table-column prop="role_id" label="角色" :formatter="checkTeacherRole" sortable>
                        <template slot-scope="scope">
                            {{ checkTeacherRole(scope.row.role_id) }}
                        </template>
                    </el-table-column>
                    <el-table-column label="带班情况" width="240" sortable>
                        <template scope="scope">
                            <span v-for="item in scope.row.cct">
                                {{ item.grade.Name + item.clazz.Name }}
                                &nbsp;&nbsp;|&nbsp;
                            </span>
                        </template>
                    </el-table-column>
                </el-table>
                <!--工具条-->
                <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="50"
                    :total="total" style="float: right">
                </el-pagination>
            </el-dialog>
        </section>
        <!-- 查看管理团队 -->
        <section>
            <el-dialog title="管理团队" width="80%" :visible.sync="TeamSubordinateTeamVisible"
                v-model="TeamSubordinateTeamVisible" :close-on-click-modal="false">
                <el-table :data="TeamSubordinateTeam" style="width: 100%; margin-bottom: 20px" border>
                    <el-table-column type="selection" width="50"> </el-table-column>
                    <el-table-column prop="name" label="团队" width="150">
                    </el-table-column>
                    <el-table-column prop="type" label="类型" width="80" sortable>
                        <template slot-scope="scope">
                            {{ getTeamType(scope.row.type) }}
                        </template>
                    </el-table-column>
                    <el-table-column prop="head_id" label="负责人" width="80">
                    </el-table-column>
                    <el-table-column prop="belongs" label="隶属">
                    </el-table-column>
                    <el-table-column prop="contact" label="联系方式">
                    </el-table-column>
                    <el-table-column prop="status" label="状态" :formatter="checkTeamStatus" sortable>
                        <template slot-scope="scope">
                            <el-tag :type="getTeamStatusTagType(scope.row.status)" disable-transitions>
                                {{ checkTeamStatus(scope.row) }}
                            </el-tag>
                        </template>
                    </el-table-column>
                    <el-table-column prop="remark" label="备注">
                    </el-table-column>
                </el-table>
            </el-dialog>
        </section>
        <!-- 查看负责人信息 -->
        <section>
            <el-dialog title="负责人" :visible.sync="TeamHeaderInfoVisible" v-model="TeamHeaderInfoVisible"
                :close-on-click-modal="false">
                <div v-if="TeamHeaderInfo">
                    <el-descriptions :column="3" border>
                        <el-descriptions-item>
                            <template slot="label">
                                工号
                            </template>
                            {{ TeamHeaderInfo.TeacherNo }}
                        </el-descriptions-item>
                        <el-descriptions-item>
                            <template slot="label">
                                姓名
                            </template>
                            {{ TeamHeaderInfo.Name }}
                        </el-descriptions-item>
                        <el-descriptions-item>
                            <template slot="label">
                                <i class="el-icon-tickets"></i>
                                职位
                            </template>
                            <el-tag size="small">
                                {{ checkTeacherRole(TeamHeaderInfo.role_id) }}
                            </el-tag>
                        </el-descriptions-item>
                        <el-descriptions-item>
                            <template slot="label">
                                联系电话
                            </template>
                            {{ TeamHeaderInfo.iNumber }}
                        </el-descriptions-item>
                    </el-descriptions>
                </div>
                <div v-else>
                    <el-empty description="暂无负责人"></el-empty>

                </div>
            </el-dialog>
        </section>
        <!-- 查看团队任务 -->
        <section>
            <el-dialog title="任务信息" width="80%" :visible.sync="TaskTableVisible" v-model="TaskTableVisible"
                :close-on-click-modal="false">
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
                <el-table :data="tasksableData" style="width: 100%; margin-bottom: 20px" border>
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
                    <el-table-column prop="status" label="状态" :formatter="checkTaskStatus" sortable>
                        <template slot-scope="scope">
                            <el-tag :type="getTaskStatusTypeTag(scope.row.status)" disable-transitions>
                                {{ checkTaskStatus(scope.row.status) }}
                            </el-tag>
                        </template>
                    </el-table-column>
                    <el-table-column prop="create_by" label="创建人" width="80" sortable>
                    </el-table-column>
                    <el-table-column prop="CreateTime" label="创建时间" :formatter="formatCreateTime" sortable>
                    </el-table-column>
                    <el-table-column prop="remark" label="备注">
                    </el-table-column>
                </el-table>
                <!--工具条-->
                <el-pagination layout="prev, pager, next" @current-change="handleCurrentChange" :page-size="50"
                    :total="total" style="float: right">
                </el-pagination>
            </el-dialog>
        </section>
    </div>

</template>

<script>
import util from "../../../util/date";
import {
    AddTeam,
    GetTeams,
    DelTeam,
    UpdateTeam,
    GetTeamHeaderInfo,
    GetAllTeams,
    GetAllTeachers,
    GetSelectTeamTasksByTeamId,
    getTeamTasksByTeamId,
    GetTasks,
    getTeamMembers,
    GetTeamSubordinateTeam
} from "../../api/api";

export default {
    data() {
        return {
            tableData: [],
            listLoading: false,
            filters: {
                name: "",
            },
            total: 0,
            page: 1,
            listLoading: false,
            sels: [], //列表选中列
            // 新增
            addForm: {
                name: '',
                type: "0",
                member_ids: [],
                task_ids: "",
                head_id: 0,
                subordinate_team_ids: [],
                belongs: "",
                contact: "",
                remark: "",
            },
            addFormRules: {
                name: [{ required: true, message: "请输入团队名称", trigger: "blur" }],
                type: [{ required: true, message: "请选择团队类型", trigger: "blur" }],
                belongs: [{ required: true, message: "须填写该团队隶属", trigger: "blur" }],
                contact: [{ required: true, message: "请输入联系方式", trigger: "blur" }],
                member_ids: [{ required: true, message: "请选择团队成员", trigger: "blur" }],
            },
            teamType: [
                {
                    value: "0000",
                    label: "网络技术系"
                },
                {
                    value: "0001",
                    label: "办公室"
                },
                {
                    value: "0102",
                    label: "网络工程教研室"
                },
                {
                    value: "0103",
                    label: "网络安全教研室"
                },
                {
                    value: "0104",
                    label: "信息工程教研室"
                },
                {
                    value: "0201",
                    label: "学工室"
                },
                {
                    value: "0",
                    label: "其他"
                },
            ],
            allTeachers: [
                {
                    label:"暂无",
                    value:0
                }
            ],
            allTeams: [],
            selectTeamTask: [
                {
                    value: 0,
                    label: "暂无"
                }
            ],
            addLoading: false,
            addFormVisible: false,
            selectTeachers: [
                {
                    value: 0,
                    label: "暂无"
                }
            ],
            // 编辑
            editFormVisible: false,
            editLoading: false,
            editForm: {
                name: '',
                type: "0",
                member_ids: [],
                head_id: 0,
                task_ids: [],
                subordinate_team_ids: [],
                belongs: "",
                contact: "",
                remark: "",
            },
            editFormRules: {
                name: [{ required: true, message: "请输入团队名称", trigger: "blur" }],
                type: [{ required: true, message: "请选择团队类型", trigger: "blur" }],
                belongs: [{ required: true, message: "须填写该团队隶属", trigger: "blur" }],
                contact: [{ required: true, message: "请输入联系方式", trigger: "blur" }],
                member_ids: [{ required: true, message: "请选择团队成员", trigger: "blur" }],
            },
            // 团队任务
            TaskTableVisible: false,
            tasksableData: [],
            getTeamTasksInfoLoading: false,
            total: 0,
            taskInfoId: 0,
            // 团队成员
            TeamTableVisible: false,
            TeamMembersableData: [],
            getTeamInfoLoading: false,
            // 团队负责人
            TeamHeaderInfo: {},
            TeamHeaderInfoVisible: false,
            // 管理团队
            TeamSubordinateTeam: [],
            TeamSubordinateTeamVisible: false
        };
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
        getTeamStatusTagType: function (status) {
            var statusTypeMap = {
                0: 'success',
                1: 'danger',
            };
            return statusTypeMap[status] || 'default';
        },
        checkTeamStatus: function (row) {
            var statusDict = {
                0: "活跃",
                1: "解散",
            };
            return statusDict[row.status] || "未知";
        },
        formatCreateTime: function (row, column) {
            return !row.CreateTime || row.CreateTime == ""
                ? ""
                : util.formatDate.format(new Date(row.CreateTime), "yyyy-MM-dd");
        },
        getTeamType(type) {
            var teamType = {
                "0000": "网络技术系",
                "0001": "办公室",
                "0102": "网络工程教研室",
                "0103": "网络安全教研室",
                "0104": "信息工程教研室",
                "0201": "学工室",
                "0": "其他",
            }
            return teamType[type];
        },
        getTaskStatusTypeTag: function (status) {
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
        checkTaskStatus: function (status) {
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
        handleCurrentChange(val) {
            this.page = val;
            this.getTeam();
        },
        //获取团队
        getTeam: function () {
            let userInfo = JSON.parse(window.localStorage.getItem('user'))
            let para = {
                page: this.page,
                key: this.filters.name,
                teamids: userInfo.team_ids
            }
            GetTeams(para).then(res => {
                this.tableData = res.data.response.data;
                this.total = res.data.response.dataCount;
            })
        },
        //显示新增界面
        addTeam() {
            GetAllTeams().then(res => {
                this.allTeams = res.data.response
            })
            GetAllTeachers().then(res => {
                this.allTeachers = res.data.response
            })
            this.addFormVisible = true;
        },
        addSubmit: function () {
            let _this = this;
            this.$refs.addForm.validate((valid) => {
                if (valid) {
                    this.$confirm("确认提交吗？", "提示", {}).then(() => {
                        this.addLoading = true;
                        let para = Object.assign({}, _this.addForm);
                        para.member_ids = para.member_ids.join(',');
                        para.subordinate_team_ids = para.subordinate_team_ids.join(',');
                        para.IsDeleted = false;
                        AddTeam(para).then((res) => {
                            if (util.isEmt.format(res)) {
                                this.addLoading = false;
                                return;
                            }
                            this.addLoading = false;
                            if (res.data.success) {
                                this.addLoading = false;
                                this.$message({
                                    message: res.data.msg,
                                    type: "success",
                                });
                                _this.$refs["addForm"].resetFields();
                                _this.addFormVisible = false;
                                _this.getTeam();
                            } else {
                                _this.$message({
                                    message: res.data.msg,
                                    type: "error",
                                });
                            }
                        });
                    });
                }
            });
        },
        //显示编辑界面
        handleEdit: function (index, row) {
            if (typeof row.member_ids === "string") {
                row.member_ids = row.member_ids.split(",").map(Number)
            }
            if (typeof row.task_ids === "string") {
                row.task_ids = row.task_ids.split(",").map(Number)
            }
            if (typeof row.subordinate_team_ids === "string") {
                row.subordinate_team_ids = row.subordinate_team_ids.split(",").map(Number)
            }
            GetSelectTeamTasksByTeamId({ id: row.Id }).then(res => {
                this.selectTeamTask = res.data.response
            })
            GetAllTeams().then(res => {
                this.allTeams = res.data.response
            })
            GetAllTeachers().then(res => {
                this.allTeachers = res.data.response
                this.selectTeachers = []
                for (let i = 0; i < row.member_ids.length; i++) {
                    let select = res.data.response.filter(item => item.value === row.member_ids[i])
                    this.selectTeachers.push(select[0]);
                }
            })

            this.editForm = Object.assign({}, row);

            this.editFormVisible = true;
        },
        //编辑
        editSubmit: function () {
            this.$refs.editForm.validate((valid) => {
                if (valid) {
                    this.$confirm("确认提交吗？", "提示", {}).then(() => {
                        this.editLoading = true;
                        //NProgress.start();
                        let para = Object.assign({}, this.editForm);
                        para.member_ids = para.member_ids.join(',');
                        para.task_ids = para.task_ids.join(',');
                        para.subordinate_team_ids = para.subordinate_team_ids.join(',');
                        UpdateTeam(para).then((res) => {
                            if (util.isEmt.format(res)) {
                                this.editLoading = false;
                                return;
                            }
                            if (res.data.success) {
                                this.editLoading = false;
                                //NProgress.done();
                                this.$message({
                                    message: res.data.msg,
                                    type: "success",
                                });
                                this.$refs["editForm"].resetFields();
                                this.editFormVisible = false;
                                this.getTeam();
                            } else {
                                this.$message({
                                    message: res.data.msg,
                                    type: "error",
                                });
                                this.editForm = {}
                                this.editFormVisible = false
                                this.editLoading = false
                            }
                        });
                    });
                }
            });
        },
        handleSelectChange(value) {
            // console.log(value);
            this.selectTeachers = []
            for (let i = 0; i < value.length; i++) {
                let select = this.allTeachers.filter(item => item.value === value[i])
                this.selectTeachers.push(select[0]);
            }
        },
        //显示任务详情界面
        lookTasksInfo: function (team_id) {
            let that = this;
            this.taskInfoId = team_id
            that.getTeamTasksInfoLoading = true;


            this.options = [];
            let para = { id: team_id };
            getTeamTasksByTeamId(para).then((res) => {
                this.tasksableData = res.data.response
                that.getTeamTasksInfoLoading = false;
            });
            this.TaskTableVisible = true;
        },
        getTaskByName: function () {
            let para = {
                page: this.page,
                key: this.filters.name,
                teamids: this.taskInfoId.toString()
            }
            GetTasks(para).then(res => {
                this.tableData = res.data.response.data;
                this.tasksableData = res.data.response.dataCount;
            })
        },
        //显示团队成员
        lookMembersInfo: function (ids) {
            this.getTeacher(ids)
            this.TeamTableVisible = true;
        },
        getTeacher(ids) {
            let para = {
                memberIds: ids
            };
            this.getTeamInfoLoading = true;

            getTeamMembers(para).then((res) => {
                this.total = res.data.response.length;
                for (let i in res.data.response) {
                    res.data.response[i].Class_ids =
                        res.data.response[i].Class_ids.split(",").map(Number);
                }
                this.TeamMembersableData = res.data.response;
                this.getTeamInfoLoading = false;
            });
        },
        checkTeacherRole: function (role_id) {
            var statusDict = {
                0: "未知",
                1: "管理员",
                2: "教师",
                3: "教务员",
                4: "负责人",
            };

            return statusDict[role_id] || "未知";
        },
        // 显示团队负责人
        lookHeaderInfo: function (id) {
            let para = {
                team_id: id
            };
            GetTeamHeaderInfo(para).then(res => {
                this.TeamHeaderInfo = res.data.response
            })
            this.TeamHeaderInfoVisible = true
        },
        // 显示管理的团队
        lookTeamsInfo: function (subordinate_team_ids) {
            let para = {
                ids: subordinate_team_ids
            }
            GetTeamSubordinateTeam(para).then(res => {
                console.log(res);
                this.TeamSubordinateTeam = res.data.response
            })
            this.TeamSubordinateTeamVisible = true
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
        handleDel: function (team_id) {
            this.listLoading = true;
            let para = {
                id: team_id
            }
            DelTeam(para).then(res => {
                if (util.isEmt.format(res)) {
                    this.listLoading = false;
                    return;
                }
                this.listLoading = false;
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
                this.getTeam();
            })
        }
    },
    mounted() {
        this.getTeam();
    },
};
</script>

<style scoped></style>