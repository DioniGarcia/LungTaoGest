<template>
  <div id="dashboard">
    <Navbar />

    <div class="wn-col col-pendientes">
      <img src="../assets/kfh2.jpg">
        <el-button class="btn-login reflex" type="primary" @click="handleLogin">Aceptar</el-button>



    </div>

    <div class="wn-col col-asignadas">
      <img src="../assets/kf1.jpg"></img>
      <el-button class="btn-login reflex" type="primary" @click="handlePrevious">Cancelar</el-button>
    </div>


    <div class="wn-col col-realizadas">
      <img src="../assets/kf1.jpg"></img>
      <el-button class="btn-login reflex" type="primary" @click="handlePrevious">Cancelar</el-button>
    </div>


  </div>



</template>



<script>
  import Navbar from './Navbar';
  import Task from './Task';
  import TaskActiva from './TaskActiva';
  import db from './firebaseInit';
  import ButtonGroup from "bootstrap-vue/es/components/button-group/button-group";
  import InputTag from 'vue-input-tag';
  import ElContainer from "element-ui/packages/container/src/main";

  export default {
    name: 'dashboard',
    data() {
      return {

        dialogVisible: false,
        dialogEditVisible: false,

        templates: [],
        tasks_sin_asignar: [],
        tasks_asignadas:   [],
        tasks_realizadas:  [],

        // Modelo de datos del form
        frm_operario: '',
        frm_fechaRealizacion: '',
        frm_asignable:   true,
        frm_titulo:        '',
        frm_descripcion:   '',
        frm_estimado:      '',
        frm_prioridad:    200,
        frm_pausable:   false,
        frm_etiquetas:     [],

        //Restablecer el modal de edición
        rst_operario:'',
        rst_fechaRealizacion:'',
        rst_asignable: true,
        rst_titulo: '',
        rst_descripcion: '',
        rst_estimado: '',
        rst_prioridad: 200,
        rst_pausable: false,
        rst_etiquetas: [],



        prioridades: [
          { value: 400, text: 'Urgente' },
          { value: 300, text: 'Alta' },
          { value: 200, text: 'Media' },
          { value: 100, text: 'Baja' },
          { value: 0,   text: 'Muy baja' },
        ],

        tagRecomendations: [],

        inputVisible: false,
        inputValue: '',
        id:'',
        loading: true,
        searchWord: '',
        currentRow: null
      }
    },
    created: function () {


      db
        .collection('sinAsignar')
        //.orderBy("prioridad","desc").orderBy("id") OLVIDAOS DE ORDERBY's => LAMBDA's
        .onSnapshot(querySnapshot => {
          this.tasks_sin_asignar.length=0
          querySnapshot.docs.map(doc => {
            const task = {
              id: doc.id,
              operario: "Se asignará durante la ejecución",
              titulo: doc.data().titulo,
              duracion: 0,
              estimado: doc.data().estimado,
              prioridad: doc.data().prioridad,
              showMore: false,
              descripcion: doc.data().descripcion,
              pausable: doc.data().pausable,
              tags: doc.data().etiquetas,
              asignable: doc.data().asignable,
              h_inicio: doc.data().h_inicio
            };

            if(task.asignable){
              this.tasks_sin_asignar.push(task);

              this.tasks_sin_asignar.sort(function(a, b) {
                return b["prioridad"] - a["prioridad"] || a["id"] - b["id"];
              });
            }

          })
        });

      db
        .collection('asignadas')
        //.orderBy("prioridad","desc").orderBy("id")
        .onSnapshot(querySnapshot => {
          this.tasks_asignadas.length=0
          var h_actual = new Date().getTime()
          querySnapshot.docs.map(doc => {
            //date_inicio = Date(doc.data().h_inicio);
            //var h_inicio=date_inicio.getTime();

            const task = {
              id: doc.id,
              operario: doc.data().operario,
              titulo: doc.data().titulo,
              duracion: 0,
              estimado: doc.data().estimado,
              prioridad: doc.data().prioridad,
              showMore: false,
              descripcion: doc.data().descripcion,
              pausable: doc.data().pausable,
              tags: doc.data().etiquetas,
              aceptada: doc.data().aceptada,
              h_inicio: doc.data().h_inicio
            };
            if (task.aceptada) {
              this.tasks_asignadas.push(task);
            }

            this.tasks_asignadas.sort(function(a, b) {
              return b["prioridad"] - a["prioridad"] || a["id"] - b["id"];
            });
          })
        });


      db.collection('finalizadas')
        //this.tasks_realizadas.length=0
        .onSnapshot(querySnapshot => {
          this.tasks_realizadas.length=0;
          querySnapshot.forEach(doc => {
            const task = {
              id: doc.id,
              operario: doc.data().operario,
              titulo: doc.data().titulo,
              duracion: Math.round((doc.data().h_fin.getTime()-doc.data().h_inicio.getTime())/60000),
              estimado: doc.data().estimado,
              prioridad: doc.data().prioridad,
              showMore: false,
              descripcion: doc.data().descripcion,
              pausable: doc.data().pausable,
              tags: doc.data().etiquetas,
              h_inicio: doc.data().h_inicio
            };
            this.tasks_realizadas.push(task);
            this.tasks_realizadas.sort(function(a, b) {
              return b["prioridad"] - a["prioridad"] || a["id"] - b["id"];
            });
        });
      });

      this.fillTemplates();

    },
    computed: {
      filteredTemplates: function(){
        return this.templates.filter((template) => {
          return template.titulo.toLowerCase().match(this.searchWord.toLowerCase())
        })
      }
    },
    methods: {
      resetFields(){
          this.frm_operario = this.rst_operario,
          this.frm_asignable = this.rst_asignable,
          this.frm_titulo = this.rst_titulo,
          this.frm_descripcion = this.rst_descripcion,
          this.frm_estimado=  this.rst_estimado,
          this.frm_prioridad=  this.rst_prioridad,
          this.frm_pausable = this.rst_pausable,
          this.frm_etiquetas = this.rst_etiquetas.slice(),
          this.frm_fechaRealizacion = this.rst_fechaRealizacion
      },
      clearFields(){
        this.cleanForm();
      },
      fillTemplates(){
        db.collection('plantillas').orderBy('titulo').get().then(querySnapshot => {
          this.loading = false;
          querySnapshot.forEach(doc => {
            const template = {
              id: doc.id,
              titulo: doc.data().titulo,
            };

            this.templates.push(template);

          });
        });
      },

      fillModal(id){

      },
      handleCurrentChange(val) {
        this.fillDataFromTemplate(val.id);

        this.currentRow = val;
      },
      querySearch(queryString, cb) {
        var tagRecomendations = this.tagRecomendations;
        var results = queryString ? tagRecomendations.filter(this.createFilter(queryString)) : tagRecomendations;
        // call callback function to return suggestions
        cb(results);
      },
      createFilter(queryString) {
        return (coso) => {
          return (coso.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0);
        };
      },
      handleSelectTag(item) {
        this.handleInputConfirm();
      },
      loadTags() {
        var tags = [];

        db.collection('etiquetas').get().then(querySnapshot => {
          querySnapshot.forEach(doc => {
            const t = {
              "value": doc.data().tag
            };
            tags.push(t);
          });
        });
        return tags;
      },

      createTask (evt) {
        evt.preventDefault()
        if (!this.frm_titulo) {
          alert('El título no puede estar vacío')
        }
        else if (!this.frm_estimado) {
          alert('El tiempo estimado no puede estar vacío')
        }
        else if (isNaN(this.frm_estimado)) {
          alert('El tiempo estimado debe ser un número entero')
        }
        else {
          this.dialogVisible = false;
          this.persistData()
        }
      },

      editTask (evt) {
        evt.preventDefault()
        if (!this.frm_titulo) {
          alert('El título no puede estar vacío')
        }
        else if (!this.frm_estimado) {
          alert('El tiempo estimado no puede estar vacío')
        }
        else if (isNaN(this.frm_estimado)) {
          alert('El tiempo estimado debe ser un número entero')
        }
        else {
          this.dialogEditVisible = false;
          this.updateTask(this.id)
        }
      },
      fillData(id){
        var opRef = db.collection("sinAsignar").doc(id.toString());

        opRef.get()
          .then(doc => {
            this.frm_operario = "No Asignada",
            this.frm_asignable = doc.data().asignable,
            this.frm_titulo = doc.data().titulo,
            this.frm_descripcion = doc.data().descripcion,
            this.frm_estimado=  doc.data().estimado,
            this.frm_prioridad=  doc.data().prioridad,
            this.frm_pausable = doc.data().pausable,
            this.frm_etiquetas = doc.data().etiquetas,
            this.frm_fechaRealizacion = doc.data().fecha_realizacion,
            this.id = doc.id,

            this.frm_operario = "No Asignada",
            this.rst_asignable = doc.data().asignable,
            this.rst_titulo = doc.data().titulo,
            this.rst_descripcion = doc.data().descripcion,
            this.rst_estimado=  doc.data().estimado,
            this.rst_prioridad=  doc.data().prioridad,
            this.rst_pausable = doc.data().pausable,
            this.rst_etiquetas = doc.data().etiquetas,
            this.rst_fechaRealizacion = doc.data().fecha_realizacion
          }).catch(function(error) {
          console.log("Error getting document:", error);
        });
      },
      fillDataFromTemplate(id){

        var opRef = db.collection("plantillas").doc(id.toString());

        opRef.get()
          .then(doc => {
            this.frm_operario = doc.data().operario,
              this.frm_asignable = doc.data().asignable,
              this.frm_titulo = doc.data().titulo,
              this.frm_descripcion = doc.data().descripcion,
              this.frm_estimado=  doc.data().estimado,
              this.frm_prioridad=  doc.data().prioridad,
              this.frm_pausable = doc.data().pausable,
              this.frm_etiquetas = doc.data().etiquetas,
              this.frm_fechaRealizacion = doc.data().fecha_realizacion,
              this.id = doc.id
          }).catch(function(error) {
          console.log("Error gettings document:", error);
        });
      },
      updateTask(id) {
        var tsRef = db.collection("sinAsignar").doc(id.toString());

        return tsRef.update({
          descripcion: this.frm_descripcion,
          estimado: parseInt(this.frm_estimado),
          etiquetas: this.frm_etiquetas,
          pausable: this.frm_pausable,
          prioridad: this.frm_prioridad,
          titulo: this.frm_titulo

        }).then(function() {
            console.log("Tarea actualizada con éxito!");
        }).then(() => {
            this.handleEdit(id)
        }).catch(function(error) {
            console.error("Error actualizando la tarea: ", error);
        });
      },
      persistData () {

        var autRef = db.collection('autoIncrems').doc('tareas');
        var transaction = db.runTransaction( async t => {
          return t.get(autRef)
            .then(doc => {

              var newTask = doc.data().tareas + 1;
              this.id = newTask;
              db.collection('sinAsignar').doc(newTask.toString()).set({

                asignable: true,
                descripcion: this.frm_descripcion,
                estimado: parseInt(this.frm_estimado),
                etiquetas: this.frm_etiquetas,
                fecha_realizacion: Date.now(),
                pausable: this.frm_pausable,
                prioridad: this.frm_prioridad,
                titulo: this.frm_titulo,
                h_inicio: 0,
                h_fin: 0,
                satisfaccion: 0,
                id: newTask,
                operario: 0,
                aceptada: false

              }).then(docRef => {
                console.log('Tarea añadida a FireBase!')

                for (var i = 0; i < this.frm_etiquetas.length; i++) {
                  var eti = this.frm_etiquetas[i].toString();

                  this.persistTag(eti);
                }
              }).then(() => {
                this.handleSubmit()

              }).catch(error => {
                console.error('Error añadiendo la tarea!',error)
              });
              t.update(autRef, {tareas: newTask});

            })
        })

      },

      deleteTask(id) {

        if (confirm('¿Estas seguro que quieres borrar la tarea con ID: ' + id + ' ?')) {
          var evalTags = [];
          var ref = db.collection('sinAsignar').doc(id.toString());

          return ref.get()
            .then(doc => {
              evalTags = doc.data().etiquetas;
              for (var i = 0; i < evalTags.length; i++) {
                this.decreaseTag(evalTags[i])
              }
            }).then(doc => {
              db.collection('sinAsignar').doc(id.toString()).delete().then(function () {
                console.log("Tarea borrada con éxito!");
              }).catch(function (error) {
                console.error("Error borrando tarea!: ", error);
              });

              this.removeFromModel(id);
            })
        }
      },

      removeFromModel(id){

        var idx = -1;
        var i = 0;
        this.tasks_sin_asignar.forEach(function(task) {
          if(task.id == id){
            idx=i;
          };
          i++;
        });
        this.tasks_sin_asignar.splice(idx,1);
      },

      decreaseTag(tag){

        var ref = db.collection('etiquetas').doc(tag);

        return ref.get()
          .then(doc => {
            var num = doc.data().veces;
            console.log(num)
            if(num > 1){
              ref.update({veces:num-1})
            }else{
              ref.delete().then(function() {
                console.log("Etiqueta borrada con éxito!");
              }).catch(function(error) {
                console.error("Error borrando etiqueta!: ", error);
              });
            }
          })

      },
      persistTag(tag){

        var refTag = db.collection('etiquetas').doc(tag);

        var getDoc = refTag.get().then(doc => {

          if (!doc.exists) {
            refTag.set({tag: tag, veces: 1})
              .then(docRef => {
                console.log('Nueva etiqueta añadida a FireBase!: '+tag)
                this.tagRecomendations = this.loadTags();
              })
              .catch(error => {console.error('Error añadiendo la etiqueta!: '+tag, error)});
          } else {
            var updateVeces = refTag.set({tag: tag, veces: doc.data().veces + 1})
              .then(docRef => {console.log('Etiqueta incrementada en FireBase!: '+tag)})
          }
        })
          .catch(err => {console.log('Error getting document', err);});
      },
      handleEdit(id) {
        var i = 0;
        for(i=0; i<this.tasks_sin_asignar.length; i++){

          if(this.tasks_sin_asignar[i].id == id){
            this.tasks_sin_asignar[i].titulo=this.frm_titulo;
            this.tasks_sin_asignar[i].operario=this.frm_operario;
            this.tasks_sin_asignar[i].pausable=this.frm_pausable;
            this.tasks_sin_asignar[i].prioridad=this.frm_prioridad;
            this.tasks_sin_asignar[i].estimado=this.frm_estimado;
            this.tasks_sin_asignar[i].descripcion=this.frm_descripcion;
            this.tasks_sin_asignar[i].etiquetas=this.frm_etiquetas;
          }
        }

        this.dialogEditVisible = false;
        this.cleanForm();
      },
      handleSubmit () {
        this.dialogVisible = false;
        this.cleanForm();
      },

      handleClose(tag) {

        this.frm_etiquetas.splice(this.frm_etiquetas.indexOf(tag), 1);
      },

      showInput() {

        this.inputVisible = true;
        this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
      },

      handleInputConfirm() {

        let inputValue = this.inputValue;
        if (inputValue) {

          this.frm_etiquetas.push(inputValue);
        }
        this.inputVisible = false;
        this.inputValue = '';
      },


      cleanForm () {
        this.frm_titulo='';
        this.frm_descripcion='';
        this.frm_estimado='';
        this.frm_prioridad=200;
        this.frm_pausable=false;
        this.frm_etiquetas=[]
        this.id = '';
      },
    },

    mounted() {
      this.tagRecomendations = this.loadTags();
    },

    components: {
      ElContainer,
      ButtonGroup,
      Navbar,
      Task,
      TaskActiva,
      'bootstrap-modal': require('vue2-bootstrap-modal'),
      InputTag
    }
  }
</script>

<style>
  div.wn-col div{
    padding-left:  10px;
    padding-top:    3px;
    padding-bottom: 3px;
  }
  div.wn-col {
    width: 33.3333%;
    float: left;
    margin-top: -9px;
    border: solid 2px white;
    height: 100%;
  }
  div.wn-col-title {
    background: cadetblue;
    color: white;
    font-size: 16px;
    padding-bottom: 2px;
  }


  div.wn-col-container::-webkit-scrollbar {
    width: 8px;
    max-width: 8px;
  }

  div.wn-col-container::-webkit-scrollbar-track {
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.3);
    border-radius: 10px;
  }

  div.wn-col-container::-webkit-scrollbar-thumb {
    border-radius: 10px;
    -webkit-box-shadow: inset 0 0 6px rgba(0,0,0,0.5);
  }


  div.wn-col-container {
    overflow-y:scroll;
    overflow-x: hidden;
    align-content: center;
    background: whitesmoke;
    padding: 0;
    margin:  0;
    height: 600px;
    width:  100%;
  }

  div.wn-task-container {
    margin-top: 3px;

    height: 80px;
  }
  div.col-pendientes {
    border-left: none;
  }
  div.col-asignadas {
    border-left: none;
    border-right: none;
  }
  div.col-realizadas {
    border-right: none;
  }


  div.wn-task-data {
    width: 80%;
    float: left;
  }

  div.wn-task-data h3 {
    font-size:     16px;
  }

  div.wn-task-data p {
    font-size:     12px;
    padding-top:    4px;
    padding-bottom: 0px;
    margin-bottom:  -6px;
  }

  div.wn-btn-div {
    width: 2px;
    float: right;
  }
  div.options {
    margin-top: 3px

  }
  .el-button {
    margin-top: 3px !important;
  }

  button.wn-menu-btn {
    border: none;
    background: whitesmoke;
    color: cadetblue;
    font-size: 12px;
    float: right;
    width: 140px;
    padding-top:    2px;
    padding-bottom: 2px;
    padding-left:  34px;
    padding-right: 34px;
    margin-top:     1px;
    margin-right:  24px;
  }

  div.wn-btn-div .wn-menu-btn {
    font-size: 12px;
    text-align: left;
    width: 20px;
    float: right;
    margin:    0;
    margin-right:  6px;
  }

  .el-tag + .el-tag {
    margin-left: 10px;
  }
  .button-new-tag {
    margin-left: 10px;
    height: 32px;
    line-height: 30px;
    padding-top: 0;
    padding-bottom: 0;
  }
  .input-new-tag {
    width: 180px;
    margin-left: 10px;
    vertical-align: bottom;
  }
  .modal-col-plantillas {
    width: 5%;
    float: left;
    border-right: 1px solid whitesmoke ;
    margin-right: 20px;
  }

  .modal-col-data {
    width: 80%;
    height: 100%;
  }

  .modal-footer {
    padding-top: 40px;
    border-top: 1px solid whitesmoke;
  }

  .resize-triggers { /*Recomendador de etiquetas*/

  }

  .fondo-buscador {
    background-color: #ffffff;
    color: #5f9ea0;

    /*border-top:    solid  2px whitesmoke;*/
    border-left:   solid   1px #5F9EA0;
    border-right:  solid   1px #5F9EA0;
    border-bottom: solid   1px #5F9EA0;
    border-bottom-radius: 50px;
    padding-left: 40px !important;
  }

  .bounce-enter-active {
    animation: bounce-in .1s;
  }
  .bounce-leave-active {
    animation: bounce-in .1s reverse;
  }
  @keyframes bounce-in {
    0% {
      opacity: 0;
      height: 20px;
    }
    50% {
      opacity: 50;
      height: 40px;
    }
    100% {
      opacity: 100;
      height: 73px;
    }
  }

  body {
    width: 100%;
    height: 100%;
    background-image: url("../assets/background.png") ;
    background-position: center center;
    background-repeat:  no-repeat;
    background-attachment: fixed;
    background-size:  cover;
    background-color: #999;
  }

  .btn-opt {
    padding-left:  30px;
    padding-right: 30px;
    margin-top: -40px;
    margin-bottom: 20px;
    margin-right: 14%;
    font-size: 21px;
    border-radius: 50px;
    background: indianred;
  }

</style>
