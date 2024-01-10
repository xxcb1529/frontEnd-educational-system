<template>
  <div class="wrapper">
    <ul class="bg-bubbles">
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
      <li></li>
    </ul>
    <div class="current-time">{{ currentTime }}</div>
    <h3 class="title">教务系统</h3>

    <el-form
      :model="ruleForm2"
      :rules="rules2"
      ref="ruleForm2"
      label-position="left"
      label-width="0px"
      class="demo-ruleForm login-container"
    >
      <el-form-item prop="account">
        <el-input
          type="text"
          v-model="ruleForm2.account"
          auto-complete="off"
          placeholder="账号"
        ></el-input>
      </el-form-item>
      <el-form-item prop="checkPass">
        <el-input
          v-model="ruleForm2.checkPass"
          auto-complete="off"
          show-password
          placeholder="密码"
        ></el-input>
      </el-form-item>
      <el-checkbox v-model="checked" checked class="remember"
        >记住密码</el-checkbox
      >

      <div style="margin-bottom: 20px" class="count-test">
        <el-radio-group @change="loginAccount" v-model="account3">
          <el-radio-button label="管理员"></el-radio-button>
          <el-radio-button label="教师"></el-radio-button>
        </el-radio-group>
      </div>
      <el-form-item style="width: 100%">
        <el-button
          type="primary"
          style="width: 100%"
          @click.native.prevent="handleSubmit2"
          :loading="logining"
        >
          {{ loginStr }}
        </el-button>
      </el-form-item>
      <el-form-item style="width: 100%">
        <el-button
          :loading="loginingMock"
          style="width: 100%"
          @click.native.prevent="handleSubmitMock"
          >学生免账号登录
        </el-button>
      </el-form-item>
    </el-form>

    <div class="subtitle">Copyright © 2023 12 Graduation Project For Lennon Xu</div>
  </div>
</template>

<script>
import {
  requestLogin,
  requestLoginMock,
  getUserByToken,
  getNavigationBar,
} from "../api/api";

import router from "@/router";
import { resetRouter, filterAsyncRouter } from "@/router/index";

export default {
  data() {
    return {
      instance: "",
      loginStr: "登录",
      logining: false,
      loginingMock: false,
      ruleForm2: {
        account: "Admin001",
        checkPass: "123456",
      },
      account3: "管理员",
      rules2: {
        account: [{ required: true, message: "请输入账号", trigger: "blur" }],
        checkPass: [{ required: true, message: "请输入密码", trigger: "blur" }],
      },
      checked: true,
      currentTime: this.getCurrentTime(),
    };
  },
  computed: {
},
  methods: {
    getCurrentTime() {
      const now = new Date();
      const hours = String(now.getHours()).padStart(2, '0');
      const minutes = String(now.getMinutes()).padStart(2, '0');
      const seconds = String(now.getSeconds()).padStart(2, '0');
      return `${hours}:${minutes}:${seconds}`;
    },
    handleReset2() {
      this.$refs.ruleForm2.resetFields();
    },
    loginAccount() {
      if (this.account3 == "管理员") {
        this.ruleForm2.account = "Admin001";
        this.ruleForm2.checkPass = "123456";
      } else if (this.account3 == "教师") {
        this.ruleForm2.account = "G2999";
        this.ruleForm2.checkPass = "G2999";
      } else {
        this.ruleForm2.account = "blogadmin";
        this.ruleForm2.checkPass = "blogadmin";
      }
    },
    //这个是用来测试 mock 的，很简单，只需要在 main.js 中开启服务即可
    handleSubmitMock(ev) {
      var _this = this;
      _this.$router.replace("/Student/SingleCourse");
    },
    openAlert(msg) {
      this.instance = this.$notify({
        title: "提示",
        message: msg,
        duration: 0,
        position: "top-left",
      });
    },
    closeAlert() {
      this.instance.close();
    },
    // 获取 Token
    handleSubmit2(ev) {
      var _this = this;
      this.$refs.ruleForm2.validate((valid) => {
        if (valid) {
          //_this.$router.replace('/table');
          this.logining = true;

          //NProgress.start();
          var loginParams = {
            name: this.ruleForm2.account,
            pass: this.ruleForm2.checkPass,
          };

          // _this.openAlert("登录中...")

          _this.loginStr = "登录中...";

          requestLogin(loginParams).then((data) => {
            if (!data.success) {
              _this.$message({
                message: data.message,
                type: "error",
              });
              _this.logining = false;
              _this.loginStr = "重新登录";
              // _this.closeAlert()
            } else {
              var token = data.token;
              _this.$store.commit("saveToken", token);

              var curTime = new Date();
              var expiredate = new Date(
                curTime.setSeconds(curTime.getSeconds() + data.expires_in)
              );
              _this.$store.commit("saveTokenExpire", expiredate);

              window.localStorage.refreshtime = expiredate;
              window.localStorage.expires_in = data.expires_in;

              _this.$notify({
                type: "success",
                message: `成功获取令牌，项目初始化中...`,
                duration: 3000,
              });

              // _this.closeAlert()
              // _this.openAlert("成功获取Token，等待服务器返回用户信息...")
              _this.loginStr = "成功获取Token，等待服务器返回用户信息...";

              _this.getUserInfoByToken(token);
            }
          });
        } else {
          console.log("error submit!!");
          return false;
        }
      });
    },
    // 获取用户数据
    getUserInfoByToken(token) {
      var _this = this;
      var loginParams = { token: token };
      getUserByToken(loginParams).then((data) => {
        if (!data.success) {
          _this.$message({
            message: data.message,
            type: "error",
          });
        } else {
          // _this.closeAlert()
          // _this.openAlert("接收到用户数据，开始初始化路由树...")
          _this.loginStr = "接收到用户数据，开始初始化路由树...";

          window.localStorage.user = JSON.stringify(data.response);
          if (data.response.uID > 0) {
            _this.GetNavigationBar(data.response.uID);
          } else {
            _this.$message({
              message: "用户数据有误！",
              type: "error",
            });
          }
        }
      });
    },
    // 获取路由树
    GetNavigationBar(uid) {
      var _this = this;
      var loginParams = { uid: uid, t: new Date() };

      getNavigationBar(loginParams).then((data) => {
        _this.logining = false;

        if (!data.success) {
          _this.$message({
            message: data.msg,
            type: "error",
          });
          _this.loginStr = "重新登录";
        } else {
          // _this.closeAlert()

          _this.$message({
            message: "后台初始化成功",
            type: "success",
          });

          let userinfo = JSON.parse(
            window.localStorage.user ? window.localStorage.user : null
          );
          _this.$notify({
            type: "success",
            message: `登录成功 \n 欢迎管理员：${
              userinfo.uRealName
            } ！Token 将在 ${
              window.localStorage.expires_in / 60 / 60
            } 小时后过期！`,
            duration: 6000,
          });

          window.localStorage.router = JSON.stringify(data.response.children);

          let getRouter = data.response.children; //后台拿到路由
          getRouter = filterAsyncRouter(getRouter); //过滤路由
          router.addRoutes(getRouter); //动态添加路由

          _this.$router.replace(
            _this.$route.query.redirect ? _this.$route.query.redirect : "/"
          );
        }
      });
    },
  },
  mounted() {
    // window.localStorage.clear()
    // 每秒更新一次当前时间
    setInterval(() => {
      this.currentTime = this.getCurrentTime();
    }, 1000);
    console.info("%c 本地缓存已清空!", "color:green");
  },
};
</script>

<style>
.login-container {
  -webkit-border-radius: 5px;
  border-radius: 5px;
  -moz-border-radius: 5px;
  background-clip: padding-box;
  margin: 15% auto;
  width: 300px;
  padding: 35px 35px 15px 35px;
  background: #fff;
  border: 1px solid #eaeaea;
  z-index: 9999;
  position: relative;
  opacity: 0.8;
}

.title {
  margin: 0px auto;
  text-align: center;
  color: #505458;
  position: absolute;
  left: 50%;
  top: 22%;
  font-size: 2rem;
  font-weight: bolder;
  font-family:'Courier New', Courier, monospace;
  transform: translateX(-50%);
}

.login-container .remember {
  margin: 0px 0px 25px 0px;
}

.wrapper.form-success .containerLogin h1 {
  -webkit-transform: translateY(85px);
  -ms-transform: translateY(85px);
  transform: translateY(85px);
}

.containerLogin {
  max-width: 600px;
  margin: 0 auto;
  padding: 80px 0;
  height: 400px;
  text-align: center;
}

.containerLogin h1 {
  font-size: 40px;
  -webkit-transition-duration: 1s;
  transition-duration: 1s;
  -webkit-transition-timing-function: ease-in-put;
  transition-timing-function: ease-in-put;
  font-weight: 200;
}

.wrapper {
  background: #f0f2f5;
  background: linear-gradient(to bottom right, #dee0e4 0%, #caced4 100%);
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.wrapper.form-success .container h1 {
  transform: translateY(85px);
}

.bg-bubbles {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
}

.bg-bubbles li {
  position: absolute;
  list-style: none;
  display: block;
  width: 40px;
  height: 40px;
  background-color: rgba(255, 255, 255, 0.15);
  bottom: -160px;
  -webkit-animation: square 25s infinite;
  animation: square 25s infinite;
  transition-timing-function: linear;
  border-radius: 8px;
}

.bg-bubbles li:nth-child(1) {
  left: 10%;
}

.bg-bubbles li:nth-child(2) {
  left: 20%;
  width: 80px;
  height: 80px;
  -webkit-animation-delay: 2s;
  animation-delay: 2s;
  -webkit-animation-duration: 17s;
  animation-duration: 17s;
}

.bg-bubbles li:nth-child(3) {
  left: 25%;
  -webkit-animation-delay: 4s;
  animation-delay: 4s;
}

.bg-bubbles li:nth-child(4) {
  left: 40%;
  width: 60px;
  height: 60px;
  -webkit-animation-duration: 22s;
  animation-duration: 22s;
  background-color: rgba(255, 255, 255, 0.25);
}

.bg-bubbles li:nth-child(5) {
  left: 70%;
}

.bg-bubbles li:nth-child(6) {
  left: 80%;
  width: 120px;
  height: 120px;
  -webkit-animation-delay: 3s;
  animation-delay: 3s;
  background-color: rgba(255, 255, 255, 0.2);
}

.bg-bubbles li:nth-child(7) {
  left: 32%;
  width: 160px;
  height: 160px;
  -webkit-animation-delay: 7s;
  animation-delay: 7s;
}

.bg-bubbles li:nth-child(8) {
  left: 55%;
  width: 20px;
  height: 20px;
  -webkit-animation-delay: 15s;
  animation-delay: 15s;
  -webkit-animation-duration: 40s;
  animation-duration: 40s;
}

.bg-bubbles li:nth-child(9) {
  left: 25%;
  width: 10px;
  height: 10px;
  -webkit-animation-delay: 2s;
  animation-delay: 2s;
  -webkit-animation-duration: 40s;
  animation-duration: 40s;
  background-color: rgba(255, 255, 255, 0.3);
}

.bg-bubbles li:nth-child(10) {
  left: 90%;
  width: 160px;
  height: 160px;
  -webkit-animation-delay: 11s;
  animation-delay: 11s;
}

@-webkit-keyframes square {
  0% {
    transform: translateY(0);
  }

  100% {
    transform: translateY(-700px) rotate(600deg);
  }
}

@keyframes square {
  0% {
    transform: translateY(0);
  }

  100% {
    transform: translateY(-700px) rotate(600deg);
  }
}
.current-time {
  position: fixed; /* 固定定位，使其可以在屏幕上自由移动 */
  top: 50%; /* 从顶部开始的距离 */
  left: 50%; /* 从左侧开始的距离 */
  transform: translate(-50%, -50%); /* 使元素在垂直和水平方向上居中 */
  font-size: 450px; /* 调整字体大小 */
  font-weight: bolder;
  color: #e9ebee; /* 设置颜色为白色 */
  width: 100%; /* 宽度撑满屏幕 */
  text-align: center; /* 文本居中 */
  z-index: 10; /* 确保时间在其他元素之上 */
}
.subtitle{
    margin: 0px auto;
  text-align: center;
  color: #505458;
  position: absolute;
  left: 50%;
  bottom: 5%;
  font-size: 1rem;
  font-weight: lighter;
  transform: translateX(-50%);
}
</style>
