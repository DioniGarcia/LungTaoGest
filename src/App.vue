<template>
  <div id="app" >
    <div>
      <router-view/>
    </div>
  </div>
  </template>


<script>
  export default {
    name: 'app',
  }
</script>

<style>

  body {
    overflow-y: hidden;
  }

  nav {
    margin-bottom: 7px;
    border-bottom: 2px solid whitesmoke ;
    box-shadow: none;
  }
</style>
