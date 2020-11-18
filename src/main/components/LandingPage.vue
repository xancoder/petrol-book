<template>
  <diV>
    <v-row>
      <v-col cols="12">
        <v-file-input
          v-model="databaseFile"
          v-on:change="loadDatabase()"
          v-on:click:clear="resetDatabase()"
          show-size
          truncate-length="42"
          prepend-icon="mdi-database"
          accept=".json"
        ></v-file-input>
      </v-col>
      <v-col cols="12">
        <v-data-table
          :headers="tableHeaders"
          :items="workingData.fuelingOperations"
          :sort-by="['date']"
          :sort-desc="[true]"
          class="elevation-1"
        >
          <template v-slot:top>
            <v-toolbar flat>
              <v-toolbar-title>Fueling Operations</v-toolbar-title>
              <v-divider class="mx-4" inset vertical></v-divider>
              <v-spacer></v-spacer>
              <v-dialog v-model="tableDialog" max-width="500px">
                <template v-slot:activator="{ on, attrs }">
                  <v-btn
                    v-bind="attrs"
                    v-on="on"
                    class="mb-2"
                    color="primary"
                    tile
                    :disabled="!databaseFile"
                  >
                    <v-icon>mdi-plus</v-icon>
                  </v-btn>
                </template>
                <v-card>
                  <v-card-title>
                    <span class="headline">{{ formTitle }}</span>
                  </v-card-title>
                  <v-card-text>
                    <v-container>
                      <v-row>
                        <v-col cols="12" sm="6" md="4">
                          <v-menu
                            v-model="menuDate"
                            :close-on-content-click="false"
                            :nudge-right="40"
                            min-width="290px"
                            offset-y
                            transition="scale-transition"
                          >
                            <template v-slot:activator="{ on, attrs }">
                              <v-text-field
                                v-model="tableEditedItem.date"
                                v-bind="attrs"
                                v-on="on"
                                label="Date"
                                prepend-icon="mdi-calendar"
                                readonly
                              ></v-text-field>
                            </template>
                            <v-date-picker
                              v-model="tableEditedItem.date"
                              @input="menuDate = false"
                            ></v-date-picker>
                          </v-menu>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-menu
                            ref="menu"
                            v-model="menuTime"
                            :close-on-content-click="false"
                            :nudge-right="40"
                            :return-value.sync="tableEditedItem.time"
                            min-width="290px"
                            offset-y
                            transition="scale-transition"
                          >
                            <template v-slot:activator="{ on, attrs }">
                              <v-text-field
                                v-model="tableEditedItem.time"
                                v-bind="attrs"
                                v-on="on"
                                label="Time"
                                prepend-icon="mdi-clock-time-four-outline"
                                readonly
                              ></v-text-field>
                            </template>
                            <v-time-picker
                              v-if="menuTime"
                              v-model="tableEditedItem.time"
                              format="24hr"
                              full-width
                              @click:minute="$refs.menu.save(tableEditedItem.time)"
                            ></v-time-picker>
                          </v-menu>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-combobox
                            v-model="tableEditedItem.petrolStation"
                            :items="petrolStations"
                            label="Petrol Station"
                          ></v-combobox>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-combobox
                            v-model="tableEditedItem.petrolType"
                            :items="petrolTypes"
                            label="Petrol Type"
                          ></v-combobox>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field
                            v-model="tableEditedItem.costs"
                            label="Costs"
                            type="number"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field
                            v-model="tableEditedItem.liquid"
                            label="Liquid"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field
                            v-model="tableEditedItem.distance"
                            label="Distance"
                          ></v-text-field>
                        </v-col>
                        <v-col cols="12" sm="6" md="4">
                          <v-text-field
                            v-model="tableEditedItem.mileage"
                            label="Mileage"
                          ></v-text-field>
                        </v-col>
                      </v-row>
                    </v-container>
                  </v-card-text>
                  <v-card-actions>
                    <v-spacer></v-spacer>
                    <v-btn
                      color="primary darken-1"
                      text
                      @click="close"
                    >
                      Cancel
                    </v-btn>
                    <v-btn
                      color="primary darken-1"
                      text
                      @click="save"
                    >
                      Save
                    </v-btn>
                  </v-card-actions>
                </v-card>
              </v-dialog>
            </v-toolbar>
          </template>
          <template v-slot:item.actions="{ item }">
            <v-icon
              class="mr-2"
              small
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
          <template v-slot:item.ppl="{ item }">
            {{ (item.costs / item.liquid).toFixed(3) }}
          </template>
          <template v-slot:item.l100="{ item }">
            {{ ((item.liquid * 100) / item.distance).toFixed(2) }}
          </template>
          <template v-slot:item.p100="{ item }">
            {{ ((item.costs / item.liquid) * ((item.liquid * 100) / item.distance)).toFixed(2) }}
          </template>
        </v-data-table>
      </v-col>
    </v-row>
  </diV>
</template>

<script>
const fs = require('fs')

export default {
  name: 'landing-page',
  components: {},
  data: () => ({
    databaseFile: null,
    tableDialog: false,
    tableHeaders: [
      {text: 'Date', value: 'date', align: 'left'},
      {text: 'Time', value: 'time', align: 'left'},
      {text: 'Petrol Station', value: 'petrolStation', align: 'left'},
      {text: 'Petrol Type', value: 'petrolType', align: 'left'},
      {text: 'Costs', value: 'costs', align: 'right'},
      {text: 'Liquid', value: 'liquid', align: 'right'},
      {text: 'Distance', value: 'distance', align: 'right'},
      {text: 'Mileage', value: 'mileage', align: 'right', sortable: false},
      {text: 'Price Per Liquid', value: 'ppl', align: 'right', sortable: false},
      {text: 'Liquid Per 100 Distance', value: 'l100', align: 'right', sortable: false},
      {text: 'Costs Per 100 Distance', value: 'p100', align: 'right', sortable: false},
      {text: 'Actions', value: 'actions', align: 'left', sortable: false}
    ],
    tableEditedIndex: -1,
    tableEditedItem: {
      date: '',
      time: '',
      petrolStation: '',
      petrolType: '',
      costs: null,
      liquid: null,
      distance: null,
      mileage: null,
      units: {
        costs: '',
        distance: '',
        liquid: ''
      }
    },
    tableDefaultItem: {
      date: '',
      time: '',
      petrolStation: '',
      petrolType: '',
      costs: null,
      liquid: null,
      distance: null,
      mileage: null,
      units: {
        costs: '',
        distance: '',
        liquid: ''
      }
    },
    menuDate: false,
    menuTime: false,
    workingData: {
      fuelingOperations: [],
      meta: {
        manufacturer: '',
        model: ''
      },
      units: {
        costs: '',
        distance: '',
        liquid: ''
      }
    },
    workingDataDefault: {
      fuelingOperations: [],
      meta: {
        manufacturer: '',
        model: ''
      },
      units: {
        costs: '',
        distance: '',
        liquid: ''
      }
    }
  }),
  created() {
    this.initialize()
  },
  mounted() {
    console.log(this.$options.name + ' mounted')
  },
  computed: {
    formTitle() {
      return this.tableEditedIndex === -1 ? 'New Item' : 'Edit Item'
    },
    petrolStations() {
      let items = []
      for (const value of this.workingData.fuelingOperations) {
        items.push(value.petrolStation)
      }
      return items
    },
    petrolTypes() {
      let items = []
      for (const value of this.workingData.fuelingOperations) {
        items.push(value.petrolType)
      }
      return items
    }
  },
  watch: {
    dialog(val) {
      val || this.close()
    }
  },
  methods: {
    initialize() {
    },
    editItem(item) {
      this.tableEditedIndex = this.workingData.fuelingOperations.indexOf(item)
      this.tableEditedItem = Object.assign({}, item)
      this.tableDialog = true
    },
    deleteItem(item) {
      const index = this.workingData.fuelingOperations.indexOf(item)
      confirm('Are you sure you want to delete this item?') && this.workingData.fuelingOperations.splice(index, 1)
      this.saveDatabase()
    },
    close() {
      this.tableDialog = false
      this.$nextTick(() => {
        this.tableEditedItem = Object.assign({}, this.tableDefaultItem)
        this.tableEditedIndex = -1
      })
    },
    save() {
      this.tableEditedItem.costs = parseFloat(this.tableEditedItem.costs)
      this.tableEditedItem.liquid = parseFloat(this.tableEditedItem.liquid)
      this.tableEditedItem.distance = parseFloat(this.tableEditedItem.distance)
      this.tableEditedItem.mileage = parseInt(this.tableEditedItem.mileage)

      if (this.tableEditedIndex > -1) {
        Object.assign(this.workingData.fuelingOperations[this.tableEditedIndex], this.tableEditedItem)
      } else {
        this.workingData.fuelingOperations.push(this.tableEditedItem)
      }
      this.close()
      this.saveDatabase()
    },
    loadDatabase() {
      if (this.databaseFile) {
        try {
          this.workingData = JSON.parse(fs.readFileSync(this.databaseFile.path, {encoding: 'utf8', flag: 'r'}))
        } catch (e) {
          alert('Failed to load the file !' + e)
        }
      }
    },
    saveDatabase: function () {
      const data = JSON.stringify(this.workingData, null, 2)
      try {
        fs.writeFileSync(this.databaseFile.path, data, 'utf-8')
      } catch (e) {
        alert('Failed to save the file !' + e)
      }
    },
    resetDatabase() {
      console.log(this.databaseFile)
      this.databaseFile = null
      this.workingData = Object.assign({}, this.workingDataDefault)
      console.log(this.databaseFile)
    }
  }
}
</script>
