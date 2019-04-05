<template>
  <v-container>
    <v-layout justify-center column>
      <v-toolbar class="elevation-1">
        <v-toolbar-title>Vixens</v-toolbar-title>
        <v-btn color="primary" fab
                               small 
                               absolute 
                               bottom 
                               right
                               @click="createNewVixen">
                               <v-icon dark>add</v-icon>
        </v-btn>
      </v-toolbar>
      <v-data-table
        :headers="headers"
        :items="vixens"
        class="elevation-1"
      >
        <template v-slot:items="props">
          <td class="text-xs-left"
              @click="editVixen(props.item)"
          >{{ props.item.id }}</td>
          <td class="text-xs-left"
              @click="editVixen(props.item)"
          >{{ props.item.name }}</td>
          <td class="text-xs-left"
              @click="editVixen(props.item)"
          >{{ props.item.image }}</td>
          <td class="text-xs-left">
            <v-btn icon 
                   outline 
                   small 
                   color="primary"
                   @click="confirmDelete(props.item)"
            >
              <v-icon small>delete_outline</v-icon>
            </v-btn>
          </td>
        </template>
      </v-data-table>
    </v-layout>

    <!-- Create a new or edit an existing vixen -->
    <v-dialog v-model="vixenDialog.show" max-width="500px" persistent>
      <content-form
        v-bind:editMode="vixenDialog.editMode"
        v-bind:initialData="vixenDialog.vixen"
        v-on:cancel="cancelVixen"
        v-on:save="saveVixen"
      />
    </v-dialog>

    <!-- Confirm to delete a vixen -->
    <v-dialog v-model="confirmDialog.show" max-width="300px" persistent>
      <confirm-dialog
        v-bind:message="confirmDialog.message"
        v-on:cancel="cancelDelete"
        v-on:confirm="deleteVixen"
      />
    </v-dialog>

  </v-container>
</template>

<script>
  import ContentForm from './ContentForm'
  import Confirm from './Confirm'
  import axios from 'axios'

  export default {
    name: "ManageContent",
    components: {
      'content-form': ContentForm,
      'confirm-dialog': Confirm
    },
    data() {
      return {
        headers: [
          { text: 'ID', value: 'id', align: 'left' },
          { text: 'Name', value: 'name', align: 'left' },
          { text: 'Image', value: 'image', align: 'left' },
          { text: 'Actions', value: 'actions', align: 'left' }
        ],
        vixens: [],
        vixenDialog: {
          show: false,
          editMode: false,
          vixen: {}
        },
        confirmDialog: {
          show: false,
          message: "Are you sure you want to delete this vixen?",
          id: -1
        }
      }
    },
    methods: {
      activateVixenDialog(vixen = {}, editMode = false) {
        this.vixenDialog.editMode = editMode;
        this.vixenDialog.vixen = vixen;
        this.vixenDialog.show = true;
      },

      createNewVixen() {
        this.activateVixenDialog();
      },
      
      cancelVixen() {
        this.vixenDialog.show = false;
      },

      confirmDelete(vixen) { // brings up a dialog
        this.confirmDialog.id = vixen.id;
        this.confirmDialog.show = true;
      },

      deleteVixen() {
        axios.delete(`http://localhost:5000/vixens/${this.confirmDialog.id}`)
          .then(res => {
            this.confirmDialog.show = false;
            this.confirmDialog.id = -1;
            this.refreshData();
            console.log(res);
          })
          .catch(err => {
            console.log("ERROR DELETING VIXEN", err.response);
          });
      },

      cancelDelete() {
        this.confirmDialog.show = false;
        this.confirmDialog.id = -1;
      },

      saveVixen() {
        this.vixenDialog.show = false;
        this.refreshData();
      },

      editVixen(vixen) {
        this.activateVixenDialog({...vixen}, true);
      },

      refreshData() {
        axios.get('http://localhost:5000/vixens')
          .then(res => {
            this.vixens = res.data;
          })
          .catch(err => {
            console.log("ERROR FETCHING VIXENS", err.response);
          });
      }
    },
    mounted() {
      this.refreshData();
    }
  }
</script>