<template>
  <div class="body">
    <el-col :span="16">
      <!-- 六个信息 -->
      <div class="num">
        <el-card shadow="hover" v-for="item in countData" :key="item.name" :body-style="{ display: 'flex', padding: 0 }"
          class="OrderCard">
          <div slot="header" class="header">
            <span>{{ item.title }}</span>
            <div style=" padding: 8px 5px;color: white;" :style="{ backgroundColor: item.color }" type="text">{{ item.unit
            }}</div>
          </div>
          <div class="footer">
            <div class="footer-top">
              <div class="important-font">{{ item.allData }}</div>
              <div class="secondary-font">{{ item.subtitle }}</div>
            </div>
            <div class="footer-bottom">
              <div><i class="icon" :class="'el-icon-' + item.icon" :style="{ background: item.color }"></i></div>
              <div>{{ item.value }}</div>
            </div>
          </div>
        </el-card>
      </div>
      <el-tabs v-model="activeName" style="background-color: white;margin: 20px 20px 20px 30px;padding: 10px 10px;"
        type="card">
        <el-tab-pane label="任务数据" name="first">
          <!-- 柱状图 -->
          <div style="height: 360px;width: 1100px;" ref="barEcharts">
            {{ initBarEcharts() }}
          </div>
        </el-tab-pane>
        <el-tab-pane label="团队数据" name="second">
          <!-- 折线图 -->
          <div style="height: 360px; width: 1100px;" ref="lineEcharts">
            {{ initLineEcharts() }}
          </div>
        </el-tab-pane>
      </el-tabs>
    </el-col>
    <el-col :span="8">
      <div class="grid-content bg-purple" style="margin: 20px 20px 0 0;">
        <!-- 首页user信息 -->
        <el-card shadow="hover">
          <div class="userCard">
            <el-avatar :size="150" :src="imgUrl"></el-avatar>
            <div class="userInfo">
              <p class="important-font">{{ userInfo.uRealName }}</p>
              <p class="secondary-font">{{ userInfo.RoleName }}</p>
            </div>
          </div>
          <div class="login-info">
            <p>上次登录时间：{{ userInfo.uLastErrTime }}</p>
          </div>
        </el-card>
        <!-- 首页表格 -->
        <el-card shadow="hover" class="tableInfo">
          <div slot="header">
            <span class="important-font">系统信息</span>
          </div>
          <div>
            <el-table :data="tableData" stripe border height="375px" style="width: 100%">
              <el-table-column prop="name" label="名称" width="180">
              </el-table-column>
              <el-table-column prop="description" label="描述"> </el-table-column>
            </el-table>
          </div>
        </el-card>
      </div>
    </el-col>
  </div>
</template>

<script>
import * as echarts from "echarts";
import { getSystemInfo } from '../../api/api'
export default {
  name: "Index",
  data() {
    return {
      activeName: 'first',
      imgUrl: require("../../assets/JWLogo.png"),
      userInfo: {},
      value: new Date(),
      tableData: [],
      countData: [
        {
          title: "任务数量",
          subtitle: "总任务数量",
          value: 1200,
          icon: "success",
          color: "#2ec7c9",
          allData: 1,
          unit: '周'
        },
        {
          title: "教师数量",
          subtitle: "总教师数量",
          value: 1200,
          icon: "star-on",
          color: "#ffb980",
          allData: 1,
          unit: '学期'
        },
        {
          title: "学生数量",
          subtitle: "总学生数量",
          value: 1200,
          icon: "s-goods",
          color: "#5ab1ef",
          allData: 1,
          unit: '年'
        },
        {
          title: "课程数量",
          subtitle: "总课程数量",
          value: 1200,
          icon: "success",
          color: "#2ec7c9",
          allData: 1,
          unit: '学期'
        },
        {
          title: "课室数量",
          subtitle: "总课室数量",
          value: 1200,
          icon: "star-on",
          color: "#ffb980",
          allData: 1,
          unit: '学期'
        },
        {
          title: "团队数量",
          subtitle: "总团队数量",
          value: 1200,
          icon: "star-on",
          color: "#ffb980",
          allData: 1,
          unit: '周'
        },
        // {
        //   title: "带班数量",
        //   subtitle: "总带班数量",
        //   value: 1200,
        //   icon: "s-goods",
        //   color: "#5ab1ef",
        //   allData:1,
        //   unit:'学期'
        // },
      ],
    };
  },
  methods: {
    //柱状图
    initBarEcharts() {
      // 新建一个promise对象
      let p = new Promise((resolve) => {
        resolve();
      });
      //然后异步执行echarts的初始化函数
      p.then(() => {
        //	此dom为echarts图标展示dom
        let myChart = echarts.init(this.$refs.barEcharts);
        let option = {
          tooltip: {},
          legend: {
            data: ["近一周"],
          },
          xAxis: {
            data: ["周一", "周二", "周三", "周四", "周五", "周六","周日"],
          },
          yAxis: {},
          series: [
            {
              name: "近一周",
              type: "bar",
              data: [5, 20, 36, 10, 10, 20,6,8],
            },
          ],
        };
        // 使用刚指定的配置项和数据显示图表。
        myChart.setOption(option);
      });
    },
    //折线图
    initLineEcharts() {
      let p = new Promise((resolve) => {
        resolve();
      });
      //然后异步执行echarts的初始化函数
      p.then(() => {
        let myChart = echarts.init(this.$refs.lineEcharts);
        let option = {
          tooltip: {},
          legend: {
            data: ["近一周"],
          },
          xAxis: {
            type: "category",
            boundaryGap: false,  // 如果你希望 x 轴的数据点贴近坐标轴两边，可以设置为 false
            data: ["周一", "周二", "周三", "周四", "周五", "周六","周日"],
          },
          yAxis: {
            type: "value",
          },
          series: [
            {
              name: "近一周",
              type: "line",  // 修改为 "line"
              data: [5, 20, 36, 10, 10, 20,6,8],
              smooth: true
            },
          ],
        };
        myChart.setOption(option);
      });
    },
  },
  mounted() {

    this.userInfo = JSON.parse(window.localStorage.getItem("user"))
    var curTime = new Date();
    if (window.localStorage.TokenExpire) {
      var expiretime = new Date(Date.parse(window.localStorage.TokenExpire));
      if (curTime >= expiretime) {
        this.$router.push("/login");
      }
    } else {
      this.$router.push("/login");
    }
    getSystemInfo().then(res => {
      for (const [key, value] of Object.entries(res.data)) {
        this.tableData.push({
          name: key,
          description: value
        });
      }
    })
  },
};
</script>

<style lang="less" scoped>
.body {
  background-color: #f2f3f6;
  margin: 0;
  width: 100%;
  height: 100%;
}

.userCard {
  height: 180px;
  display: flex;
  border-bottom: 1px solid #dcdfe6;
  border-radius: 2px;
}

.userInfo {
  width: auto;
  padding: 6% 0 0 6%;
}

.important-font {
  font-size: 24px;
}

.secondary-font {
  color: #909399;
}

.login-info {
  height: 40px;
  text-align: left;
  color: #909399;
  line-height: 40px;
}

.tableInfo {
  margin: 20px 0 0 0;
}

.OrderCard {
  margin: 20px 0 0px 30px;
  font-size: 14px;
  width: 30%;

  .header {
    height: 5px;
    line-height: 5px;
    display: flex;
    align-items: center;
    justify-content: space-between;
  }

  .footer {
    display: flex;
    justify-content: space-between;
    align-items: center;
    width: 100%;
    padding: 20px 20px;

    .footer-top {
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
      height: 70px;

    }

    .footer-bottom {
      display: flex;
      flex-direction: column;
      justify-content: space-between;
      align-items: center;
      height: 70px;

      i {
        font-size: 30px;
        color: #fff;
        text-align: center;
      }
    }

  }

}

.el-card {
  border: none;
}

.num {
  display: flex;
  flex-wrap: wrap;
}

.graph {
  margin: 0;
}

.el-calendar {
  height: 265px;
}
</style>
