<template>
    <div class="body">
        <el-row v-if="taskData">
            <el-col :span="8" v-for="(item, index) in taskData" :key="item.Id">
                <div class="task-content">
                    <div class="team-name">{{ item.team_name }}</div>
                    <div>
                        <div class="children" v-for="(task, index) in item.children" :key="task.Id">
                            <div class="left">
                                <div style="width: 6px;height: 16px;margin-right: 0.5rem;"
                                    :style="{ backgroundColor: getColor(task.status) }"></div>{{ task.name }}
                            </div>
                            <div class="status">
                                <i class="fa" :style="{ color: getColor(task.status) }"
                                    :class="getStatusTagType(task.status)"></i>
                                {{ checkStatus(task.status) }}
                            </div>
                        </div>
                    </div>
                </div>
            </el-col>
        </el-row>
        <div v-else class="nothing">
            <img src="../../assets/暂无公告.png" alt="">
            No Team Participation
        </div>
    </div>
</template>
<script>
import { color } from "echarts/lib/export";
import {
    GetTasksByTeamIds
} from "../../api/api";
export default {
    data() {
        return {
            taskData: []
        }
    },
    methods: {
        getTeamTask() {
            let userInfo = JSON.parse(window.localStorage.getItem('user'))
            let para = {
                ids: userInfo.team_ids
            };
            GetTasksByTeamIds(para).then((res) => {
                this.taskData = res.data.response
                console.log(res);
            }).catch(err => {
                console.log(err);
            })
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
    },
    mounted() {
        this.getTeamTask()
    }
}
</script>
<style scoped lang="less">
.nothing{
    position: fixed;
    bottom: 50%;
    right: 50%;
    transform: translate(50%,50%);
    display: flex;
    flex-direction: column;
    align-items: center;
    width: 200px;
    height: 220px;
    font-size: 20px;
    font-weight: 600;
    font-family:'Times New Roman', Times, serif;
    color: rgba(0, 0, 0, .2);
    img{
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
}</style>