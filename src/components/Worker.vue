<template>
  <div class="wn-task-data">
    <h3>
      {{this.nombre}} {{this.apellidos}}

    </h3>
    <p><b>Disciplina: </b> {{this.disciplina}} </p>
    <p><b>Fecha nacimiento:</b> {{this.nacimiento}}</p>

    <div v-if="showMore" class="more-info">
      <p><b>Títulos: </b></p>
      <div class="el-tag" v-for="t in this.tags"><p>{{t}}</p></div>
    </div>

    <div class="end-line">
      <p></p>
    </div>

  </div>
</template>

<script>
  export default {
    name: "task",

    props: [
      'id',
      'nombre',
      'apellidos',
      'tags',
      'nacimiento',
      'disciplina',
      'showMore',
      'conectado'
    ]
  }
</script>

<style scoped>

  b {
    color: indianred;
  }

  .end-line {
    border-bottom: 3px solid white;
    width: 160%;
    margin-top: 10px;
    margin-left: -20px;
  }

  .el-tag {
    margin-top:  12px;
    padding-top: 0px;
    background: indianred;
  }

  .el-tag p {
    padding: 0;
    font-size: 10px;
    color: rgba(205, 92, 92, 0.15);
  }

  div.more-info {
    padding-left: 0;
    margin-left: 0;
    margin-top: 14px;
    border-top:  1px solid indianred;
    width: 120%;
  }

  .wn-task-data {
    padding-left:  0;
    padding-right: 0;
  }




</style>
