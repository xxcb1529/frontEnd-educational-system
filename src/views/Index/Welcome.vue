<template>
  <div style="margin-top: 30px">
    <GrowCard />
  </div>
</template>

<script>
import GrowCard from './components/GrowCard.vue';
export default {
  name: "Welcome",
  components: {
    GrowCard  // 局部注册子组件
  },
  mounted() {
    var curTime = new Date();
    if (window.localStorage.TokenExpire) {
      var expiretime = new Date(Date.parse(window.localStorage.TokenExpire));
      if (curTime >= expiretime) {
        this.$router.push("/login");
      }
    } else {
      this.$router.push("/login");
    }
  },
};
</script>

<style scoped></style>
