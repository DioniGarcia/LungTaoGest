<template>
    <div id="gOperarios">
      <Navbar />

      <div class="wn-col col-pendientes">
        <div class="wn-col-title">Lista Alumnos Actuales
          <el-button @click="inputVisible=false,cleanForm(),dialogVisible = true" class="wn-menu-btn">Nuevo alumno</el-button>
        </div>

          <div class="wn-col-container">
            <div v-for="operario in alumnos" v-bind:key="operario.id" class="wn-task-container">

              <Worker
                :id=operario.id
                :nombre=operario.nombre
                :apellidos=operario.apellidos
                :nacimiento=operario.nacimiento
                :disciplina=operario.disciplina
                :tags=operario.tags
                :show-more=operario.showMore
                :conectado=operario.conectado
              />

              <div class="wn-btn-div">
                <button @click="deleteWorker(operario.id)" class="wn-menu-btn"><i class="fa fa-close " aria-hidden="true"></i></button>
                <button @click="inputVisible=false,fillData(operario.id), dialogEditVisible = true" class="wn-menu-btn"><i class="fa fa-edit " aria-hidden="true"></i></button>
                <button v-if="!operario.showMore" @click="operario.showMore=true" class="wn-menu-btn"><i class="fa fa-eye " aria-hidden="true"></i></button>
                <button v-if="operario.showMore" @click="operario.showMore=false" class="wn-menu-btn"><i class="fa fa-eye-slash " aria-hidden="true"></i></button>
              </div>

            </div>
        </div>
      </div>
      <!-- Modal NUEVO Alumno -->
      <el-dialog
        title="Alta nuevo alumno"
        :visible.sync="dialogVisible"
        width="45%"
        >

        <el-form ref="modal_new_worker"  label-position="left" label-width="120px">
          <el-form-item label="Nombre:"  required >
            <el-input  type="text" v-model="frm_nombre" placeholder="Nombre del alumno"></el-input>
          </el-form-item>
          <el-form-item label="Apellidos:"  required >
            <el-input  type="text" v-model="frm_apellidos" placeholder="Apellidos del alumno"></el-input>
          </el-form-item>
          <el-form-item label="Fecha de nacimiento:"  required >
            <el-input  type="text" v-model="frm_born" placeholder="dd/mm/aaaa"></el-input>
          </el-form-item>
          <el-form-item label="DNI:"  required >
            <el-input  type="text" v-model="frm_dni" placeholder="12345678A"></el-input>
          </el-form-item>
          <el-form-item label="Disciplina:"  required >
            <el-input  type="text" v-model="frm_disc" placeholder="Kung-fú"></el-input>
          </el-form-item>
          <el-form-item label="Nº de licencia:"  required >
          <el-input  type="text" v-model="frm_license" placeholder="123456"></el-input>
          </el-form-item>
          <el-form-item label="Fecha de última licencia:"  required >
            <el-input  type="text" v-model="frm_date_ll" placeholder="dd/mm/aaaa"></el-input>
          </el-form-item>
          <el-form-item label="Grado:"  required >
          <el-input  type="text" v-model="frm_grade" placeholder="Titulación"></el-input>
        </el-form-item>
          <el-form-item label="Cuota (euros):"  required >
            <el-input  type="text" v-model="frm_payment" placeholder="45"></el-input>
          </el-form-item>



          <el-form-item label="Títulos:"  required>
            <el-tag
              :key="tag"
              v-for="tag in frm_etiquetas"
              closable
              :disable-transitions="false"
              @close="handleClose(tag)">
              {{tag}}
            </el-tag>

            <el-autocomplete
              style="width: 23%; font-size: 10px"
              class="input-new-tag"
              v-if="inputVisible"
              v-model="inputValue"
              ref="saveTagInput"
              :fetch-suggestions="querySearch"
              placeholder="Nombre nueva etiqueta"
              @keyup.enter.native="handleInputConfirm"
              @select="handleSelectTag">
            </el-autocomplete>

            <el-button v-else class="button-new-tag" size="small" @click="showInput">+ Nuevo tag</el-button>
          </el-form-item>
        </el-form>

        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogVisible = false">Cancelar</el-button>
          <el-button type="primary" @click="createWorker">Crear</el-button>
        </span>
      </el-dialog>
      <!-- FIN: Modal NUEVO Operario-->

      <!-- Modal EDITAR Operario-->
      <el-dialog
        title="Editar operario"
        :visible.sync="dialogEditVisible"
        width="65%">

        <el-form ref="modal_edit_worker" label-position="left" label-width="120px">
          <el-form-item label="Nombre:"  required >
            <el-input  type="text" v-model="frm_nombre" placeholder="Nombre del operario"></el-input>
          </el-form-item>
          <el-form-item label="Apellidos:"  required >
            <el-input  type="text" v-model="frm_apellidos" placeholder="Apellidos del operario"></el-input>
          </el-form-item>

          <el-form-item label="Etiquetas:"  required>
            <el-tag
              :key="tag"
              v-for="tag in frm_etiquetas"
              closable
              :disable-transitions="false"
              @close="handleClose(tag)">
              {{tag}}
            </el-tag>

            <el-autocomplete
              style="width: 23%; font-size: 10px"
              class="input-new-tag"
              v-if="inputVisible"
              v-model="inputValue"
              ref="saveTagInput"
              :fetch-suggestions="querySearch"
              placeholder="Nombre nueva etiqueta"
              @keyup.enter.native="handleInputConfirm"
              @select="handleSelectTag">
            </el-autocomplete>

            <el-button v-else class="button-new-tag" size="small" @click="showInput">+ Nuevo tag</el-button>
          </el-form-item>

        </el-form>

        <span slot="footer" class="dialog-footer">
          <el-button @click="dialogEditVisible = false">Cancelar</el-button>
          <el-button type="primary" @click="editWorker">Guardar</el-button>
        </span>
      </el-dialog>
      <!-- FIN: Modal EDITAR Operario-->

    </div>
</template>

<script>
  import Navbar from './Navbar';
  import Worker from './Worker';
  import db from './firebaseInit'
  import ButtonGroup from "bootstrap-vue/es/components/button-group/button-group";
  import InputTag from 'vue-input-tag';

  export default {
    name: 'gOperarios',
    data() {
      return {

        dialogVisible: false,
        dialogEditVisible: false,

        alumnos: [],

        //Form crear
        frm_nombre: '',
        frm_apellidos: '',
        frm_born: '',
        frm_disc: '',
        frm_dni:'',
        frm_license: '',
        frm_date_ll: '',
        frm_grade: '',
        frm_payment: '',
        frm_etiquetas:     [],
        id:'',

        tagRecomendations: [],

        inputVisible: false,
        inputValue: '',

        conectado: '',
        loading: true
      }
    },


    // RELLENAR EL ARRAY DE OPERARIOS CON LOS DATOS DE FIREBASE
    created() {
      db
        .collection('alumnos')
        .onSnapshot(querySnapshot => {
          this.loading = false;
          this.alumnos.length=0
          querySnapshot.forEach(doc => {
            const operario = {
              nombre: doc.data().nombre,
              apellidos: doc.data().apellidos,
              nacimiento: doc.data().nacimiento,
              dni: doc.data().dni,
              disciplina: doc.data().disciplina,
              nLicencia: doc.data().nLicencia,
              dLLicencia: doc.data().dLLicencia,
              grado: doc.data().grado,
              cuota: doc.data().cuota,
              id: doc.id, // El autoincrement de FB
              tags: doc.data().etiquetas,
              showMore: false,
              //conectado: doc.data().conectado
            }

            this.alumnos.push(operario)

           /* this.alumnos.sort(function(a, b) {
              return !a.conectado && b.conectado
            });*/
          });
        });

    },
    components: {
      ButtonGroup,
      Navbar,
      Worker,
      'bootstrap-modal': require('vue2-bootstrap-modal'),
      InputTag
    },

    methods: {

      createWorker(evt) {
        evt.preventDefault()
        if (!this.frm_nombre) {
          alert('El nombre no puede estar vacío')
        }
        else if (!this.frm_apellidos) {
          alert('Los apellidos no puden estar vacios')
        }
        else if (!this.frm_born) {
          alert('Se requiere una fecha de nacimiento')
        }
        else if (!this.frm_dni) {
          alert('Se requiere poner un dni')
        }
        else if (!this.frm_disc) {
          alert('Se requiere poner una disciplina')
        }
        else if (!this.frm_grade) {
          alert('Se requiere poner un grado')
        }
        else if (!this.frm_payment) {
          alert('Debe introducirse una cuota mensual')
        }
        else if (isNaN(this.frm_payment)) {
          alert('La cuota mensual debe ser numérica')
        }
        else {
          this.dialogVisible = false;
          this.persistData()
        }
      },

      fillData(id){

        var opRef = db.collection("alumnos").doc(id.toString());

        opRef.get()
          .then(doc => {
            this.frm_nombre = doc.data().nombre;
            this.frm_apellidos = doc.data().apellidos;
            this.frm_dni = doc.data().dni;
            this.frm_born = doc.data().nacimiento;
            this.frm_payment = doc.data().cuota;
            this.frm_license = doc.data().nLicencia;
            this.frm_date_ll = doc.data().dLLicencia;
            this.id = id;
            this.frm_etiquetas= doc.data().etiquetas;

        }).catch(function(error) {
          console.log("Error gettings document:", error);
        });
      },


      editWorker(evt){
        evt.preventDefault()

        if (!this.frm_nombre) {
          alert('El nombre no puede estar vacío')
        }
        else if (!this.frm_apellidos) {
          alert('Los apellidos no pueden estar vacios')
        }


        else {
          this.dialogEditVisible = false;
          this.updateEmployee(this.id)
        }
      },

      persistData() {

        var autRef = db.collection('autoIncrems').doc('alumnos');
        var transaction = db.runTransaction( async t => {
          return t.get(autRef)
            .then(doc => {

              var newOp = doc.data().alumnos + 1;

              console.log('Valor: '+newOp)

              this.id = newOp;

              db.collection('alumnos').doc(newOp.toString()).set({

                nombre: this.frm_nombre,
                apellidos: this.frm_apellidos,
                dni: this.frm_dni,
                disciplina: this.frm_disc,
                nacimiento: this.frm_born,
                nLicencia: this.frm_license,
                dLLicencia: this.frm_date_ll,
                grado: this.frm_grade,
                cuota: this.frm_payment,

                id: doc.id, // El autoincrement de FB
                tags: this.frm_etiquetas,
                showMore: false,


                etiquetas: this.frm_etiquetas,
                id: newOp,
                conectado: false,

              }).then(docRef => {
                  console.log('Alumno añadido a FireBase!')

                  for (var i = 0; i < this.frm_etiquetas.length; i++) {
                    var eti = this.frm_etiquetas[i].toString();

                    this.persistTag(eti);
                  }
                }).then(() => {
                  this.handleSubmit()

                }).catch(error => {
                  console.error('Error añadiendo el alumnoo!', error)
                });
              t.update(autRef, {alumnos: newOp});
            })
        })
      },

      handleSubmit() {

        this.cleanForm();
      },

      handleEdit(id) {
        var i = 0;
        for(i=0; i<this.alumnos.length; i++){
          if(this.alumnos[i].id == id){
            this.alumnos[i].nombre=this.frm_nombre;
            this.alumnos[i].apellidos=this.frm_apellidos;

            this.alumnos[i].etiquetas=this.frm_etiquetas;
          }
        }
        this.dialogEditVisible = false;
        this.cleanForm();
      },

      cleanForm() {
        console.log('faig')
        this.frm_nombre = '';
        this.frm_apellidos = '';
        this.frm_disc = '';
        this.frm_grade = '';
        this.frm_dni = '';
        this.frm_born = '';
        this.frm_payment = '';
        this.frm_license = '';
        this.frm_date_ll = '';
        this.id = '';
        this.frm_etiquetas= [];

      },

      removeFromModel(id){
        var idx = -1;
        var i = 0;
        this.alumnos.forEach(function(alumno) {
          if(alumno.id == id){
            idx=i;
          };
          i++;
        });
        this.alumnos.splice(idx,1);
      },

      deleteWorker(id) {
        if (confirm('¿Estas seguro que quieres borrar el alumno: ' + id + ' ?')) {
          var evalTags = [];
          var ref = db.collection('alumnos').doc(id.toString());

          return ref.get()
            .then(doc => {
              evalTags = doc.data().etiquetas;
              for (var i = 0; i < evalTags.length; i++) {
                this.decreaseTag(evalTags[i])
              }
            }).then(doc => {
              db.collection('alumnos').doc(id.toString()).delete().then(function () {
                console.log("Operario borrado con éxito!");
              }).catch(function (error) {
                console.error("Error borrando operario!: ", error);
              });
              this.removeFromModel(id);
            })
        }
      },

      updateEmployee(id) {
        var operarioRef = db.collection("alumnos").doc(id.toString());

        return operarioRef.update({
          nombre: this.frm_nombre,
          apellidos: this.frm_apellidos,

          etiquetas: this.frm_etiquetas
        })
          .then(function() {
            console.log("Operario actualizado con éxito!");
          }).then(() => {
            this.handleEdit(this.id)
          }).catch(function(error) {
            console.error("Error actualizando operario: ", error);
          });
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
          console.log('input_value:'+inputValue)
          this.frm_etiquetas.push(inputValue);
        }
        this.inputVisible = false;
        this.inputValue = '';
      },

    },
    mounted() {
      this.tagRecomendations = this.loadTags();
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
    background: indianred;
    color: white;
    font-size: 16px;
    padding-bottom: 2px;
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

  button.wn-menu-btn {
    border: none;
    background: whitesmoke;
    color: indianred;
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
    width: 90px;
    margin-left: 10px;
    vertical-align: bottom;
  }

</style>

