<template>
  <el-button @click="loginOut" type="info">退出</el-button>
</template>
<script>
export default {
  methods: {
    loginOut() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    }
  }
}
</script>
<style lang="less" scoped>
</style>
