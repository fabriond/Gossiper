<template>
  <div id="app">
    <div class="navbar">
      <a href="/" class="left">Gossiper</a>
      <div>O site para comentar nas despesas públicas do portal da transparência</div>
    </div>
    <router-view/>
  </div>
</template>

<script>
export default {
  name: 'App'
}
</script>

<style>
#app {
  font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 10px;
}
body{margin: 0;}
.navbar {
  overflow: hidden;
  background-color: #333;
  position: fixed;
  top: 0;
  width: 100%;
  z-index: 500;
}
.navbar a, .navbar div{
  font-family: 'Gill Sans', 'Gill Sans MT', Calibri, 'Trebuchet MS', sans-serif;
}
.navbar a {
  display: block;
  color: #f2f2f2;
  text-align: center;
  padding: 14px 16px;
  text-decoration: none;
  font-size: 30px;
  background-color: #42b983;
}
.navbar div {
  display: block;
  color: #f2f2f2;
  text-align: center;
  padding: 14px 16px;
  text-decoration: none;
  font-size: 30px;
}
.right{
  float: right;
}
.left{
  float: left;
}
.navbar a:hover {
  background: #ddd;
  color: black;
}
</style>
