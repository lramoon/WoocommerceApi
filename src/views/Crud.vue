<template>
  <v-data-table
    :headers="headers"
    :items="entradas"
    sort-by="id"
    class="elevation-1"
  >
    <template v-slot:top>
      <v-toolbar
        flat
      >
        <v-toolbar-name>My CRUD</v-toolbar-name>
        <v-divider
          class="mx-4"
          inset
          vertical
        ></v-divider>
        <v-spacer></v-spacer>
        <v-dialog
          v-model="dialog"
          max-width="500px"
        >
          <template v-slot:activator="{ on, attrs }">
            <v-btn
              color="primary"
              dark
              class="mb-2"
              v-bind="attrs"
              v-on="on"
            >
              New Item
            </v-btn>
          </template>
          <v-card>
            <v-card-name>
              <span class="headline">{{ formname }}</span>
            </v-card-name>

            <v-card-text>
              <v-container>
                <v-row>
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.name"
                      label="Dessert name"
                    ></v-text-field>
                  </v-col>
                  <!-- <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.id"
                      label="id"
                    ></v-text-field>
                  </v-col> -->
                  <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.content"
                      label="content (g)"
                    ></v-text-field>
                  </v-col>
                  <!-- <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.date"
                      label="date (g)"
                    ></v-text-field>
                  </v-col> -->
                  <!-- <v-col
                    cols="12"
                    sm="6"
                    md="4"
                  >
                    <v-text-field
                      v-model="editedItem.status"
                      label="status (g)"
                    ></v-text-field>
                  </v-col> -->
                </v-row>
              </v-container>
            </v-card-text>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn
                color="blue darken-1"
                text
                @click="close"
              >
                Cancel
              </v-btn>
              <v-btn
                color="blue darken-1"
                text
                @click="save"
              >
                Save
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
        <v-dialog v-model="dialogDelete" max-width="500px">
          <v-card>
            <v-card-name class="headline">Are you sure you want to delete this item?</v-card-name>
            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="blue darken-1" text @click="closeDelete">Cancel</v-btn>
              <v-btn color="blue darken-1" text @click="deleteItemConfirm">OK</v-btn>
              <v-spacer></v-spacer>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-toolbar>
    </template>
    <template v-slot:item.actions="{item}">
      <v-icon
        small
        class="mr-2"
        @click="editItem(item)"
      >
        mdi-pencil
      </v-icon>
      <v-icon
        small
        @click="deleteItem(item)"
      >
        mdi-delete
      </v-icon>
    </template>
    <template v-slot:no-data>
      <v-btn
        color="primary"
        @click="initialize"
      >
        Reset
      </v-btn>
    </template>
  </v-data-table>
</template>

<script>
  export default {
    data: () => ({

      config:{
    headers: {
    Authorization: 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJpc3MiOiJodHRwOlwvXC9sb2NhbGhvc3RcL3dvb2NvbW1lcmNlIiwiaWF0IjoxNjE2OTAxNjM0LCJuYmYiOjE2MTY5MDE2MzQsImV4cCI6MTYxNzUwNjQzNCwiZGF0YSI6eyJ1c2VyIjp7ImlkIjoiMSJ9fX0.ZdvN5iE5qXxhL0WmC8BNnAkLLg0vro_Y9zaYXBYdtNs'
             }
        },
      dialog: false,
      dialogDelete: false,
      headers: [
        {
          text: 'Dessert (100g serving)',
          align: 'start',
          sortable: false,
          value: 'name',
        },
        { text: 'id', value: 'id' },
        { text: 'content (g)', value: 'content' },
        { text: 'date (g)', value: 'date' },
        { text: 'status (g)', value: 'status' },
        { text: 'Actions', value: 'actions', sortable: false },
      ],
      entradas: [],
      editedIndex: -1,
      editedItem: {
        name: '',
        id: '',
        content: '',
        date: '',
        status: '',
      },
      defaultItem: {
        name: '',
        id: '',
        content: '',
        date: '',
        status: '',
      },
    }),

    computed: {
      formname () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      },
    },

    watch: {
      dialog (val) {
        val || this.close()
      },
      dialogDelete (val) {
        val || this.closeDelete()
      },
    },

    created () {
      this.initialize()
    },

    methods: {

        // Para limpiar <p> del content
        limpiar(value){
        return value.replace(/<\/?[^>]+(>|$)/g, "")
        },

    async initialize () {

        try{

                const EntradasDB = await this.axios.get('/wc/v3/products', this.config)

                console.log(EntradasDB.data)

                await EntradasDB.data.forEach(element => {
                    let item = {}

                item.id = element.id;
                item.name = this.limpiar(element.name);
                item.content = this.limpiar(element.description);
                item.date = element.date;
                item.status = element.status;

                this.entradas.push(item);

                });

            } catch (error) {
                console.log(error)
            }

      },

      editItem (item) {
        this.editedIndex = this.entradas.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

    deleteItem (item) {
        this.editedIndex = this.entradas.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialogDelete = true


      },

    async deleteItemConfirm () {
      Object.assign(this.entradas[this.editedIndex], this.editedItem)
       const respuesta = this.entradas.splice(this.editedIndex, 1)
        if(respuesta){
       await this.axios.delete(`/wc/v3/products/${this.editedItem.id}`, this.config);
        this.closeDelete()
        }


      },

      close () {
        this.dialog = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })
      },

     closeDelete () {
        this.dialogDelete = false
        this.$nextTick(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        })

      },

      async save () {

        if (this.editedIndex > -1) {
    // Editar
    Object.assign(this.entradas[this.editedIndex], this.editedItem)
    console.log(this.editedItem);
    try {

      await this.axios.post(`/wc/v3/products/${this.editedItem.id}`, this.editedItem, this.config);

    } catch (error) {

      console.log(error)

    }



  } else {
    // Agregar
    // console.log(this.editedItem);
    const entrada = {
      name: this.editedItem.name,
      content: this.editedItem.content,
      status: 'publish'
    }
    try {
      const EntradaNueva = await this.axios.post('/wc/v3/products',entrada, this.config);
      console.log(EntradaNueva.data);
      this.editedItem.id = EntradaNueva.data.id;
      this.editedItem.date = EntradaNueva.data.date;
      this.editedItem.status = EntradaNueva.data.status;
    } catch (error) {
      console.log(error);
    }
    this.entradas.push(this.editedItem)

        }
        this.close()
      },
    },
  }
</script>