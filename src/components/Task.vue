<template>
    <div class="wn-task-data">
      <h3>
        <i v-if="this.prioridad==400" class="fa fa-angle-double-up hot-icon" aria-hidden="true"></i>
        <i v-if="this.prioridad==300" class="fa fa-angle-up hot-icon" aria-hidden="true"></i>
        <i v-if="this.prioridad==200" class="fa fa-angle-up mid-icon" aria-hidden="true"></i>
        <i v-if="this.prioridad==100" class="fa fa-angle-down low-icon" aria-hidden="true"></i>
        <i v-if="this.prioridad==0"   class="fa fa-angle-double-down muy-low-icon" aria-hidden="true"></i>
        {{this.titulo}}
      </h3>

      <p><b>Operario:</b>  {{this.operario}}</p>
      <p><b>Duración:</b>
        <span v-if="this.duracion <= this.estimado">{{this.duracion}}</span>
        <span v-if="this.duracion > this.estimado && this.duracion < this.estimado*2" style="color: darkorange">{{this.duracion}}</span>
        <span v-if="this.duracion > this.estimado*2" style="color: red">{{this.duracion}}</span>
        / {{this.estimado}} min</p>

      <div v-if="showMore" class="more-info">
        <p><b>Id:</b> {{this.id}}</p>
        <p><b>Pausable:</b> {{this.pausable}}</p>
        <p><b>Descipción:</b>  {{this.descripcion}}</p>
        <p><b>Etiquetas:</b></p>
        <div class="el-tag" v-for="t in this.tags"><p>{{t}}</p></div>
        <p v-if="this.tags.length == 0">No hay etiquetas asignadas</p>
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
          'titulo',
          'operario',
          'duracion',
          'estimado',
          'prioridad',
          'showMore',
          'descripcion',
          'pausable',
          'tags',
          'h_inicio'
        ],
    }


</script>

<style scoped>

  b {
    color: cadetblue;
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
    background: cadetblue;
  }

  .el-tag p {
    padding: 0;
    font-size: 10px;
    color: whitesmoke;
  }

  div.more-info {
    padding-left: 0;
    margin-left: 0;
    margin-top: 14px;
    border-top: 1px solid cadetblue;
    width: 120%;
  }

  .wn-task-data {
    padding-left:  0;
    padding-right: 0;
  }

  .hot-icon {
    color: #F5F5F5;
    background: #db5352;
    border-radius: 6px;
    padding-left: 4px;
    padding-right: 4px;
    margin-right: 4px;
  }

  .mid-icon {
    color: #F5F5F5;
    background: #dbba5c;
    border-radius: 6px;
    padding-left: 4px;
    padding-right: 4px;
    margin-right: 4px;
  }

  .low-icon {
    color: #F5F5F5;
    background: #89dbbc;
    border-radius: 6px;
    padding-left: 4px;
    padding-right: 4px;
    margin-right: 4px;
  }

  .muy-low-icon {
    color: #F5F5F5;
    background: rgb(188, 196, 194);
    border-radius: 6px;
    padding-left: 4px;
    padding-right: 4px;
    margin-right: 4px;
  }

</style>
