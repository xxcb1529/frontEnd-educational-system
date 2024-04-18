<template>
    <div>
        <el-tabs class="tab" type="border-card">
            <el-tab-pane label="参与团队">
                <el-form :model="createForm" label-width="80px" :rules="createFormRules" ref="createForm">
                    <el-form-item label="任务名" prop="name">
                        <el-input v-model="createForm.name" auto-complete="off"></el-input>
                    </el-form-item>
                    <el-form-item label="选择团队" prop="assign_team_ids">
                        <el-select v-model="createForm.assign_team_ids" multiple placeholder="请选择">
                            <el-option v-for="item in teams" :key="item.value" :label="item.label" :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="优先级" prop="priority">
                        <el-select v-model="createForm.priority" clearable placeholder="请选择">
                            <el-option v-for="item in prioritys" :key="item.value" :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="审批员" prop="approver_id">
                        <el-select v-model="createForm.approver_id" clearable placeholder="请选择">
                            <el-option v-for="item in allTeachers" :key="item.value" :label="item.label"
                                :value="item.value">
                            </el-option>
                        </el-select>
                    </el-form-item>
                    <el-form-item label="开始时间" prop="start_time">
                        <el-date-picker v-model="createForm.start_time" type="datetime" placeholder="选择开始日期时间"
                            align="right" :picker-options="pickerOptions1">
                        </el-date-picker>
                    </el-form-item>
                    <el-form-item label="结束时间" prop="end_time">
                        <el-date-picker v-model="createForm.end_time" type="datetime" placeholder="选择日期时间" align="right"
                            :picker-options="pickerOptions2">
                        </el-date-picker>
                    </el-form-item>
                    <!-- <el-form-item label="限制时间" prop="date">
                        <div class="block">
                            <el-date-picker v-model="createForm.date" type="daterange" align="right" unlink-panels
                                range-separator="至" start-placeholder="开始日期" end-placeholder="结束日期"
                                :picker-options="pickerOptions">
                            </el-date-picker>
                        </div>
                    </el-form-item> -->
                    <el-form-item label="备注" prop="remark">
                        <el-input type="textarea" :rows="1" placeholder="请输入内容" v-model="createForm.remark">
                        </el-input>
                    </el-form-item>
                    <el-form-item label="详情描述" prop="description">
                        <el-input type="textarea" :rows="5" placeholder="请输入内容" v-model="createForm.description">
                        </el-input>
                    </el-form-item>
                    <el-form-item>
                        <el-button type="primary" @click="submitForm('createForm')">立即创建</el-button>
                        <el-button @click="resetForm('createForm')">重置</el-button>
                    </el-form-item>
                </el-form>
            </el-tab-pane>
            <el-tab-pane label="个人">个人</el-tab-pane>
        </el-tabs>
    </div>
</template>
<script>
import { GetJoinTeams, AddTask, GetAllTeachers } from "../../api/api";

export default {
    data() {
        return {
            createForm: {
                name: '',
                assign_team_ids: [],
                priority: '',
                start_time: '',
                end_time: '',
                remark: '无',
                description: '无',
                approver_id: 0
            },
            addLoading: false,
            prioritys: [{
                value: 'S',
                label: 'S级'
            }, {
                value: 'A',
                label: 'A级'
            }, {
                value: 'B',
                label: 'B级'
            }, {
                value: 'C',
                label: 'C级'
            }],
            teams: [],
            pickerOptions1: {
                shortcuts: [{
                    text: '今天',
                    onClick(picker) {
                        picker.$emit('pick', new Date());
                    }
                }, {
                    text: '昨天',
                    onClick(picker) {
                        const date = new Date();
                        date.setTime(date.getTime() - 3600 * 1000 * 24);
                        picker.$emit('pick', date);
                    }
                }, {
                    text: '一周前',
                    onClick(picker) {
                        const date = new Date();
                        date.setTime(date.getTime() - 3600 * 1000 * 24 * 7);
                        picker.$emit('pick', date);
                    }
                },
                {
                    text: '半个月前',
                    onClick(picker) {
                        const date = new Date();
                        date.setTime(date.getTime() - 3600 * 1000 * 24 * 15);
                        picker.$emit('pick', date);
                    }
                },
                {
                    text: '三十天前',
                    onClick(picker) {
                        const date = new Date();
                        date.setTime(date.getTime() - 3600 * 1000 * 24 * 30);
                        picker.$emit('pick', date);
                    }
                }]
            },
            pickerOptions2: {
                shortcuts: [{
                    text: '今天',
                    onClick(picker) {
                        picker.$emit('pick', new Date());
                    }
                }, {
                    text: '明天',
                    onClick(picker) {
                        const date = new Date();
                        date.setTime(date.getTime() + 3600 * 1000 * 24);
                        picker.$emit('pick', date);
                    }
                },
                {
                    text: '一周后',
                    onClick(picker) {
                        const date = new Date();
                        date.setTime(date.getTime() + 3600 * 1000 * 24 * 7);
                        picker.$emit('pick', date);
                    }
                },
                {
                    text: '半个月后',
                    onClick(picker) {
                        const date = new Date();
                        date.setTime(date.getTime() + 3600 * 1000 * 24 * 30);
                        picker.$emit('pick', date);
                    }
                },
                {
                    text: '三十天后',
                    onClick(picker) {
                        const date = new Date();
                        date.setTime(date.getTime() + 3600 * 1000 * 24 * 30);
                        picker.$emit('pick', date);
                    }
                }
                ]
            },
            createFormRules: {
                name: [
                    { required: true, message: '请输入任务名称', trigger: 'blur' },
                ],
                assign_team_ids: [
                    { required: true, message: '请至少选择一个派发团队', trigger: 'change' }
                ],
                approver_id: [
                    { required: true, message: '请选择审批人', trigger: 'change' }
                ],
                start_time: [
                    { type: 'date', required: true, message: '请选择开始时间', trigger: 'change' }
                ],
                end_time: [
                    { type: 'date', required: true, message: '请选择结束时间', trigger: 'change' }
                ],
                priority: [
                    { required: true, message: '请选择任务的优先级', trigger: 'change' }
                ]
            },
            allTeachers: [
                {
                    label: "请选择",
                    value: 0
                }
            ]
        }
    },
    methods: {
        submitForm(formName) {
            this.$refs[formName].validate((valid) => {
                if (valid) {
                    this.$confirm("确认提交吗？", "提示", {}).then(() => {
                        this.addLoading = true;
                        let para = Object.assign({}, this.createForm);
                        para.assign_team_ids = para.assign_team_ids.join(',');
                        console.log(para);
                        AddTask(para).then(res => {
                            console.log(res);
                            this.addLoading = false
                            if (res.data.success) {
                                this.$message({
                                    message: "添加成功",
                                    type: "success",
                                });
                            } else {
                                this.$message({
                                    message: res.data.msg,
                                    type: "error",
                                });
                            }
                        })
                    })
                } else {
                    console.log('error submit!!');
                    return false;
                }
            });
        },
        resetForm(formName) {
            this.$refs[formName].resetFields();
        },
        getFormInfo() {
            let userInfo = JSON.parse(window.localStorage.getItem('user'))
            let para = {
                ids: userInfo.team_ids,
                tid: userInfo.uID
            }
            GetJoinTeams(para).then(res => {
                this.teams = res.data.response
                console.log(res);
            })
        }
    },
    mounted() {
        this.getFormInfo()
        GetAllTeachers().then(res => {
            this.allTeachers = res.data.response
        })
    }
}
</script>
<style scoped lang="less">
.tab {}
</style>