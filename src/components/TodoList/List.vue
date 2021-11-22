<template>
  <v-main class="list">
    <h3 class="text-h3 font-weight-medium mb-5">To Do List</h3>
    <v-card>
      <v-card-title>
        <v-text-field
            v-model="search"
            append-icon="mdi-magnify"
            label="Search"
            single-line
            hide-details
        ></v-text-field>
        <v-spacer></v-spacer>
        <v-btn color="success" dark @click="dialog = true">Tambah</v-btn>
      </v-card-title>
      <v-data-table :headers="headers" :items="todos" :search="search" :single-expand="expand" :expanded.sync="expanded" show-expand class="elevation-1" item-key="note">
        <template v-slot:[`item.actions`]="{ item }">
          <v-icon small class="mr-2" color="red" @click="editItem(item)">mdi-pencil</v-icon>
          <v-icon small class="mr-2" color="green" @click="deleteItem(item)">mdi-delete</v-icon>
        </template>

        <template v-slot:[`item.priority`]="{ item }">
          <v-icon v-if="item.priority == 'Penting'" @click="snackbarpenting = true" color="black">mdi-information</v-icon>
          <v-icon v-if="item.priority == 'Biasa'" @click="snackbarbiasa = true" color="black">mdi-information</v-icon>
          <v-icon v-if="item.priority == 'Tidak Penting'" @click="snackbartidakpenting = true" color="black">mdi-information</v-icon>
        </template>

        <template v-slot:expanded-item="{ headers, item }">
          <td :colspan="headers.length" v-bind:style="{ padding: 0 }">

            <v-toolbar v-if="item.priority == 'Penting'" color="red">
              <font color="white">
                {{ item.note }}
              </font>
            </v-toolbar>

            <v-toolbar v-if="item.priority == 'Biasa'" color="green">
              <font color="white">
                {{ item.note }}
              </font>
            </v-toolbar>

            <v-toolbar v-if="item.priority == 'Tidak Penting'" color="blue">
              <font color="white">
                {{ item.note }}
              </font>
            </v-toolbar>

          </td>
        </template>
      </v-data-table>
    </v-card>

    <v-snackbar v-model="snackbarpenting" :timeout="timeout" color="red" top right>Penting</v-snackbar>
    <v-snackbar v-model="snackbarbiasa" :timeout="timeout" color="green" top left>Biasa</v-snackbar>
    <v-snackbar v-model="snackbartidakpenting" :timeout="timeout" color="blue" bottom center>Tidak Penting</v-snackbar>

    <v-dialog v-model="dialog" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Form Todo</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-text-field
                v-model="formTodo.task"
                label="Task"
                required
            ></v-text-field>

            <v-select
                v-model="formTodo.priority"
                :items="['Penting', 'Biasa', 'Tidak Penting']"
                label="Priority"
                required
            ></v-select>

            <v-textarea
                v-model="formTodo.note"
                label="Note"
                required
            ></v-textarea>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="cancel">Cancel</v-btn>
          <v-btn color="blue darken-1" text @click="save">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialogdelete" max-width="600px">
      <v-card>
        <v-card-title class="headline justify-center">
          Apakah Anda Yakin Ingin Menghapus Item Ini?
        </v-card-title>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="canceldelete">Cancel</v-btn>
          <v-btn color="blue darken-1" text @click="savedelete">Hapus</v-btn>
          <v-spacer></v-spacer>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <v-dialog v-model="dialogedit" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Edit Todo</span>
        </v-card-title>
        <v-card-text>
          <v-container>
            <v-text-field
                v-model="formTodo.task"
                label="Task"
                required
            ></v-text-field>

            <v-select
                v-model="formTodo.priority"
                :items="['Penting', 'Biasa', 'Tidak Penting']"
                label="Priority"
                required
            ></v-select>

            <v-textarea
                v-model="formTodo.note"
                label="Note"
                required
            ></v-textarea>
          </v-container>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="blue darken-1" text @click="canceledit">Cancel</v-btn>
          <v-btn color="blue darken-1" text @click="saveedit">Save</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

  </v-main>
</template>

<script>
export default {
  name: "List",
  data() {
    return {
      search: null,
      dialog: false,
      dialogdelete: false,
      tempindex: -1,
      snackbarpenting: false,
      snackbarbiasa: false,
      snackbartidakpenting: false,
      timeout: 1000,
      headers: [
        {
          text: "Task",
          align: "start",
          sortable: true,
          value: "task",
        },
        { text: "Priority", value: "priority" },
        { text: "Note", value: "note" },
        { text: "Actions", value: "actions" },
      ],
      todos: [
        {
          task: "Coding",
          priority: "Penting",
          note: "Code for your life",
        },
        {
          task: "Gaming",
          priority: "Tidak Penting",
          note: "Wasting time",
        },
        {
          task: "Masak",
          priority: "Biasa",
          note: "Indomie saat coding terbaik gan",
        },
      ],
      formTodo: { task: null, priority: null, note: null },
    };
  },
  methods: {
    save() {
      this.todos.push(this.formTodo);
      this.resetForm();
      this.dialog = false;
    },
    cancel() {
      this.resetForm();
      this.dialog = false;
    },
    deleteItem(item){
      this.tempindex = this.todos.indexOf(item);
      this.formTodo = Object.assign({}, item);
      this.dialogdelete = true;
    },
    canceldelete() {
      this.dialogdelete = false;
    },
    savedelete() {
      this.todos.splice(this.tempindex, 1)
      this.closeDelete()
    },
    closeDelete () {
      this.dialogdelete = false
      this.$nextTick(() => {
        this.formTodo = Object.assign({}, this.resetForm)
        this.tempindex = -1
      });
    },
    editItem (item) {
      this.temp = this.todos.indexOf(item)
      this.formTodo = Object.assign({}, item)
      this.dialogedit = true
    },
    canceledit() {
      this.resetForm();
      this.dialogedit = false;
    },
    saveedit() {
      Object.assign(this.todos[this.temp], this.formTodo)
      this.resetForm()
      this.dialogedit = false;
    },
    resetForm() {
      this.formTodo = { task: null, priority: null, note: null };
    },
  },
};
</script>