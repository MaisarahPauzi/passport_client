<template>
  <div>
    <v-toolbar flat color="white">
      <v-toolbar-title>My Products</v-toolbar-title>
      <v-divider
        class="mx-2"
        inset
        vertical
      ></v-divider>
      <v-spacer></v-spacer>
      <v-dialog v-model="dialog" max-width="500px">
        <template v-slot:activator="{ on }">
          <v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
        </template>
        <v-card>
          <v-card-title>
            <span class="headline">{{ formTitle }}</span>
          </v-card-title>

          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedItem.name" label="Product name"></v-text-field>
                </v-flex>
                <v-flex xs12 sm6 md4>
                  <v-text-field v-model="editedItem.price" label="price"></v-text-field>
                </v-flex>
                
              </v-layout>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="blue darken-1" flat @click="close">Cancel</v-btn>
            <v-btn color="blue darken-1" flat @click="save">Save</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-toolbar>
    <v-data-table
      :headers="headers"
      :items="products"
      class="elevation-1"
    >
      <template v-slot:items="props">
        <td>{{ props.item.name }}</td>
        <td class="text-xs-right">{{ props.item.price }}</td>
        <td class="justify-center layout px-0">
          <v-icon
            small
            class="mr-2"
            @click="editItem(props.item)"
          >
            edit
          </v-icon>
          <v-icon
            small
            @click="deleteItem(props.item)"
          >
            delete
          </v-icon>
        </td>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">Reset</v-btn>
      </template>
    </v-data-table>
  </div>
</template>

<script>
import axios from 'axios';
export default {
    data: () => ({
      dialog: false,
      headers: [
        {
          text: 'Name',
          align: 'left',
          sortable: false,
          value: 'name'
        },
        { text: 'Price', value: 'price' },
        { text: 'Actions', value: 'name', sortable: false }
      ],
      products: [],
      editedIndex: -1,
      editedItem: {
        name: '',
        price: 0
        
      },
      defaultItem: {
        name: '',
        price: 0
      }
    }),

    computed: {
      
      loginToken () {
        return this.$store.state.loginToken
      },
      formTitle () {
        return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
      }
    },

    watch: {
      dialog (val) {
        val || this.close()
      }
    },

    created () {
      axios.get('http://localhost:8000/api/products', {
                     'headers':{
                         'Authorization':'Bearer '+ this.loginToken
                     }
                 }).then(response => {
                            console.log(response.data.data);
                            this.products = response.data.data;
                        })
                        .catch(error => {
                            console.log(error);
                        });
    },

    methods: {
      initialize () {
      },

      editItem (item) {


        this.editedIndex = this.products.indexOf(item)
        this.editedItem = Object.assign({}, item)
        this.dialog = true
      },

      deleteItem (item) {
        const index = this.products.indexOf(item);
        
        axios.delete('http://localhost:8000/api/products/'+item.id, 
            {
              'headers':{
                  'Accept':'application/json',
                  'Authorization':'Bearer '+ this.loginToken
              }
                 }).then(response => {
                            console.log(response);
                            
                        })
                        .catch(error => {
                            console.log(error);
                        });
        this.products.splice(index, 1);
      },

      close () {
        this.dialog = false
        setTimeout(() => {
          this.editedItem = Object.assign({}, this.defaultItem)
          this.editedIndex = -1
        }, 300)
      },

      save () {
        if (this.editedIndex > -1) {
          //edited product
          Object.assign(this.products[this.editedIndex], this.editedItem);
          axios.put('http://localhost:8000/api/products/'+this.editedItem.id, 
          {
            'name': this.editedItem.name,
            'price': this.editedItem.price          
            },
            {
              'headers':{
                  'Authorization':'Bearer '+ this.loginToken
              }
                 }).then(response => {
                            console.log(response);
                            
                        })
                        .catch(error => {
                            console.log(error);
                        });

        } else {
          //new product
          this.products.push(this.editedItem);
          axios.post('http://localhost:8000/api/products', {
            'name': this.editedItem.name,
            'price': this.editedItem.price          
            },
            {
              'headers':{
                  'Authorization':'Bearer '+ this.loginToken
              }
                 }).then(response => {
                            console.log(response);
                            
                        })
                        .catch(error => {
                            console.log(error);
                        });

        }
        this.close()
      }
    }
  }
</script>