<template>
<!-- eslint-disable max-len -->
  <div>
    <!-- Navigation -->
    <nav class="navbar navbar-expand-lg bg-secondary text-uppercase fixed-top" id="mainNav">
      <div class="container">
        <a class="navbar-brand js-scroll-trigger" href="/"><img style="width: 100px;" src="../static/img/proacademy.png"> admin panel</a>
        <button class="navbar-toggler navbar-toggler-right text-uppercase font-weight-bold bg-primary text-white rounded" type="button" data-toggle="collapse" data-target="#navbarResponsive" aria-controls="navbarResponsive" aria-expanded="false" aria-label="Toggle navigation">
          Menu
          <i class="fas fa-bars"></i>
        </button>
        <div class="collapse navbar-collapse" id="navbarResponsive">
          <ul class="navbar-nav ml-auto">
            <li class="nav-item mx-0 mx-lg-1">
              <a class="nav-link py-3 px-0 px-lg-3 rounded js-scroll-trigger" href="#portfolio">Clients</a>
            </li>
            <li class="nav-item mx-0 mx-lg-1">
              <a class="nav-link py-3 px-0 px-lg-3 rounded js-scroll-trigger" href="/about">Teachers</a>
            </li>
            <li class="nav-item mx-0 mx-lg-1">
              <a class="nav-link py-3 px-0 px-lg-3 rounded js-scroll-trigger" href="#contact">Groups</a>
            </li>
          </ul>
        </div>
      </div>
    </nav>
    <!--All clients section-->
    <div style="height: 200px;"></div>
    <b-row class="mb-5">
      <b-col lg="2" class=" mb-4" style="margin-left:70px;">
        <b-form-group class="mb-0">
            <b-input-group size="sm">
              <b-form-input
                v-model="filter"
                type="search"
                id="filterInput"
                placeholder="Type to search"
              ></b-form-input>
              <b-input-group-append>
                <b-button :disabled="!filter" @click="filter = ''">Clear</b-button>
              </b-input-group-append>
            </b-input-group>
        </b-form-group>
      </b-col>
      <b-col lg="2">
        <div>
          <b-form-checkbox @change.native="level ? level=false : level=true">filter by level</b-form-checkbox>
          <b-form-input @change.native="levelItems" id="range-1" v-model="value" type="range" min="1" max="4"></b-form-input>
          <div class="mt-2">Level: <strong>{{ value }}</strong></div>
        </div>
      </b-col>
      <b-col>
        <b-form-group label="Show only selected groups:">
          <b-form-checkbox-group
            @change.native="ageGroupItems"
            id="checkbox-group-1"
            v-model="selected"
            :options="options"
            name="flavour-1"
          ></b-form-checkbox-group>
        </b-form-group>
      </b-col>
      <b-col lg="2">
        <b-button @click="clearFilters()" variant="warning">Clear filters</b-button>
      </b-col>
    </b-row>
    <!--Table-->
    <b-table
      :items="items"
      :fields="fields"
      :filter="filter"
      :filterIncludedFields="filterOn"
      @filtered="onFiltered"
      striped
      responsive="sm"
      class="table-custom-style">
        <template v-slot:cell(details)="row">
            <b-button size="sm" variant="info" @click="row.toggleDetails" class="mr-2">
              <span class="pr-2 pl-2">Details</span>
            </b-button>

        </template>
        <template v-slot:cell(edit)="row">
            <b-button size="sm"  @click="row.toggleDetails" class="mr-2">
              <span class="pr-2 pl-2">Edit</span>
            </b-button>
        </template>
        <template v-slot:cell(delete)="row">
            <b-button size="sm" variant="dark" @click="row.toggleDetails" class="mr-2">
              Delete
            </b-button>
        </template>

        <template v-slot:row-details="row">
            <b-card>
              <b-row class="mb-2">
                <b-col sm="3" class="text-sm-right"><b>Age:</b></b-col>
                <b-col>{{ row.item.age }}</b-col>
              </b-row>

              <b-row class="mb-2">
                  <b-col sm="3" class="text-sm-right"><b>Age group:</b></b-col>
                  <b-col>{{ row.item.age_group }}</b-col>
                </b-row>

              <b-row class="mb-2">
                <b-col sm="3" class="text-sm-right"><b>Level:</b></b-col>
                <b-col>{{ row.item.level }}</b-col>
              </b-row>

              <b-button size="sm" @click="row.toggleDetails">Hide Details</b-button>
            </b-card>
        </template>
    </b-table>
    <div style="height: 100px;"></div>


  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'Admin',

  beforeCreate() {
    const path = 'http://127.0.0.1:5000/admin';
    axios.get(path).then((response) => {
      // eslint-disable-next-line
      console.log(response.data)
      this.items = response.data;
      this.copyItems = response.data;
    }).catch((error) => {
      // eslint-disable-next-line
      console.error(error);
    });
  },

  data() {
    return {
      value: 2,

      filter: null,
      filterOn: [],
      level: false,
      levelClients: [],
      options: [
        { text: 'Dorosli', value: 'dorosli' },
        { text: 'Dzieci', value: 'dzieci' },
        { text: 'Mlodziez', value: 'mlodziez' },
      ],
      selected: [],

      fields: ['id', 'name', 'surname', 'age_group', 'details', 'delete'],
      copyItems: [],
      items: [],

      filtered: [],
    };
  },

  methods: {
    getMessage() {
      const path = 'http://127.0.0.1:5000/admin';
      axios.get(path)
        .then((response) => {
          this.items = response.body;
          this.copyItems = response.body;
          // eslint-disable-next-line
          console.log(this.items);
          // eslint-disable-next-line
          console.log(this.copyItems);
        })
        .catch((error) => {
          // eslint-disable-next-line
          console.error(error);
        });
    },

    clearFilters() {
      this.items = this.copyItems;
    },

    ageGroupItems() {
      if (this.selected.length > 0) {
        this.items = this.copyItems;
        const checkedGroups = [];
        for (let i = 0; i < this.selected.length; i += 1) {
          let checked = [];
          checked = this.items.filter(item => item.age_group === this.selected[i]);
          if (checked.length > 0) {
            for (let j = 0; i < checked.length; j += 1) {
              checkedGroups.push(checked[j]);
            }
          }
        }
        this.items = checkedGroups;
      } else {
        this.items = this.copyItems;
      }
    },

    levelItems() {
      if (this.level) {
        this.items = this.copyItems;
        this.levelClients = this.items.filter(item => item.level === this.value);
        this.items = this.levelClients;
      } else {
        this.items = this.copyItems;
      }
      // eslint-disable-next-line
      console.log("level",this.copyItems);
      // eslint-disable-next-line
      console.log("items",this.items);
    },

    onFiltered(filteredItems) {
      this.totalRows = filteredItems.length;
      this.currentPage = 1;
    },
  },

  watch: {
    level() {
      this.levelItems();
    },
  },
};
</script>
