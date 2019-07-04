<template>
  <div id="app">
    <p class="link">掘金地址：<a href="https://juejin.im/user/5ad0a12d5188255c620f7fc9">_南风的主页</a></p>
    <p class="link">github地址：<a href="https://github.com/lihao336991">李浩的github</a><span class="center">如果对您有帮助，不妨给个star ^_^ </span></p>
    <HelloWorld/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
  margin-top: 60px;
}
.link{
  font-size: 13px;
  margin-left: 10px;
}
.link a{
  color: #5272FF;
}
.link .center{
  margin-left: 10px;
}
</style>
