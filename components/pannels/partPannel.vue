<template>
  <div class="component">
    <div
      v-if="part"
      class="mt-3"
    >
      <my-alert
        ref="alert"
        class="stickyAlert"
      />
      <h2>Part file upload</h2>
      <b-form>
        <b-form-group
          label="OBJ file upload:"
          label-cols-sm="4"
        >
          <b-form-file
            v-model="objFile"
            :placeholder="placeHolder"
            drop-placeholder="Drop file here..."
            accept=".obj"
            :state="part.ObjFileExcists"
            @input="submitFile"
          />
        </b-form-group>
      </b-form>
      <h2>Part edit</h2>
      <b-form
        class="part-edit"
        @click="partForViewport.part_connection = zeroSelection;"
      >
        <b-form-group
          label-class="x"
          label="X offset"
          label-cols-sm="4"
        >
          <b-form-input
            id="x_offset"
            v-model="part.x_offset"
            type="number"
            step="0.1"
          />
        </b-form-group>
        <b-form-group
          label-class="y"
          label="Y offset"
          label-cols-sm="4"
        >
          <b-form-input
            id="y_offset"
            v-model="part.y_offset"
            type="number"
            step="0.1"
          />
        </b-form-group>
        <b-form-group
          label-class="z"
          label="Z offset"
          label-cols-sm="4"
        >
          <b-form-input
            id="z_offset"
            v-model="part.z_offset"
            type="number"
            step="0.1"
          />
        </b-form-group>
        <b-form-group
          label-class="x"
          label="X angle offset"
          label-cols-sm="4"
        >
          <b-form-input
            id="x_angle_offset"
            v-model="part.x_angle_offset"
            type="number"
            step="0.1"
          />
        </b-form-group>
        <b-form-group
          label-class="y"
          label="Y angle offset"
          label-cols-sm="4"
        >
          <b-form-input
            id="y_angle_offset"
            v-model="part.y_angle_offset"
            type="number"
            step="0.1"
          />
        </b-form-group>
        <b-form-group
          label-class="z"
          label="Z angle offset"
          label-cols-sm="4"
        >
          <b-form-input
            id="z_angle_offset"
            v-model="part.z_angle_offset"
            type="number"
            step="0.1"
          />
        </b-form-group>
        <b-form-group
          v-if="connectionTypes"
          label="Fullfiled connection:"
          label-cols-sm="4"
        >
          <b-form-select
            id="connection"
            v-model="part.connection_id"
          >
            <b-form-select-option :value="null">
              <div class="">
                Null
              </div>
            </b-form-select-option>
            <b-form-select-option
              v-for="connection in connectionTypes"
              :key="connection.id"
              :value="connection.id"
            >
              <div class="">
                {{ connection.name }}
              </div>
            </b-form-select-option>
          </b-form-select>
          <div class="my-2">
            <b-button
              variant="success"
              @click="$refs.connectionForm.open(null)"
            >
              Create new
            </b-button>
            <b-button
              variant="info"
              @click="$refs.connectionForm.open(part.connection_id)"
            >
              Edit (Selected)
            </b-button>
            <b-button
              variant="danger"
              @click="deleteConnection()"
            >
              Delete (Selected)
            </b-button>
          </div>
        </b-form-group>
        <b-button
          id="submitPart"
          variant="success"
          @click="onSubmit"
        >
          Save changes
        </b-button>
      </b-form>
      <h2>Part dimentions</h2>
      <part-dimention-edit :part="part" />
      <h2>Needed Connection edit</h2>
      <part-connection-edit
        :id="id"
        :connection-types="connectionTypes"
        @selected-connection="(selection) => partForViewport.part_connection = selection"
      />
      <connection-form
        ref="connectionForm"
        @done="(id) => {getConnectionOptions(); part.connection_id = id}"
      />
    </div>
  </div>
</template>

<script>
import PartConnectionEdit from '../edit/partConnectionEdit.vue'
import partDimentionEdit from '../edit/partDimentionEdit.vue'
import ConnectionForm from '../modals/connectionForm.vue'
export default {
  name: "PartPannel",
  components: { partDimentionEdit, PartConnectionEdit, ConnectionForm },
  props:{
    id: {
      type: [Number, String],
      required: true
    },
  },
  emits: ['changed'],
  data(){
    return{
      part: null,
      partForViewport: null,
      objFile: null,
      placeHolder: "Choose a file or drop it here...",
      connectionTypes: [],
      connectionEditId: null,
      zeroSelection:{
        x_angle_offset: 0,
        y_angle_offset: 0,
        z_angle_offset: 0,
        x_offset: 0,
        y_offset: 0,
        z_offset: 0,
        editing: false
      }
    }
  },
  watch:{
    partForViewport: {
      handler() {
        this.changed()
      },
      deep: true
    },
  },
  created(){
    this.getConnectionOptions()
    this.getParts()
  },
  methods:{
    async getParts(){
      this.$axios.get("/api/part/"+this.id)
        .then(response => {
          this.part = response.data
          this.partForViewport = {...this.part, part:this.part, part_connection: this.zeroSelection, isPartSetup:true}
        })

    },
    getConnectionOptions(){
      this.$axios.get("/api/connection")
        .then(response => {
          this.connectionTypes = response.data
        })
    },
    changed(){
      this.$emit("changed", this.partForViewport)
    },
    submitFile(){
      if(this.objFile === null) return

      let formData = new FormData();
      formData.append("objFile", this.objFile);
      this.$axios.post("/api/part/"+this.id+"/objFile", formData, {
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      })
        .then(() => {
          this.changed()
          this.placeHolder = this.objFile.name
          this.objFile = null
          this.part.ObjFileExcists = true
          this.$parent.$refs.alert.setAlert(".obj file uploaded successfully", "success")
        })
        .catch((error) => {
          this.placeHolder = this.objFile.name
          this.objFile = null
          this.$refs.alert.parseError(error)
        })
    },
    onSubmit(){
      this.$axios.put("/api/part/"+this.id, this.part)
        .then((message) => {
          this.$refs.alert.parseSuccess(message)
        })
        .catch((error) => {
          this.$refs.alert.parseError(error)
        })
    },
    deleteConnection(){
      this.$axios.delete("/api/connection/"+this.part.connection_id)
        .then((message) => {
          this.$refs.alert.parseSuccess(message)
          this.getConnectionOptions();
          this.part.connection_id=null;
        })
        .catch((error) => {
          this.$refs.alert.parseError(error)
        })
    },
  }
}
</script>

<style scoped>
    .component{
        min-height: 400px;
    }
</style>
<style>
    legend.x{
        background: lightcoral;
    }
    legend.y{
        background: lightgreen;
    }
    legend.z{
        background: lightblue;
    }
</style>
