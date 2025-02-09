<template>
  <el-card class="account-container">
    <h1>联系作者</h1>
    <ul>
      <li>我的邮箱：u97966979@163.com</li>
      <li>QQ技术交流群：819182010</li>
    </ul>
  </el-card>
</template>

<script setup>
const goJuejin = () => {
  console.log('goJuejin')
  window.open('https://juejin.cn/book/6933939264455442444', 'target')
}
</script>
<style scoped>
  .course {
    font-weight: bold;
    display: inline-block;
  }
  .course li {
    line-height: 36px;
    color: #409eff;
    cursor: pointer;
  }
</style>