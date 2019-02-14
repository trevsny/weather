<template>
  <div id="app">
    <div class="container-fluid">
      <b-row>
        <b-col class="mt-4">
          <h1 class="">Let's get some weather!</h1>
        </b-col>
      </b-row>
      <b-form @submit.prevent="searchCity()">
        <b-row class="mt-4">
          <b-col class="col-md-9 col-lg-5 mx-auto">
            <b-input-group>
              <b-form-input
              v-model="search"
              type = "text"
              placeholder = "Type city">
              </b-form-input>
              <b-input-group-append>
                <b-button type = "submit">Search</b-button>    
              </b-input-group-append>                
            </b-input-group>
          </b-col>
        </b-row>      
        <b-row class="mt-4 mb-4">
          <b-col class="mx-auto">
            <b-form-group label="Need fahrenheit?"  label-for = "checkGroup" class="h6">
              <b-form-checkbox-group id="checkGroup">
                  <input
                    v-model="selected"
                    id="customCheck1"
                    type= "checkbox"
                  >
                <label for = "customCheck1">Yes, I'm American</label>
              </b-form-checkbox-group>
            </b-form-group>
          </b-col>
        </b-row>
      </b-form>  
      <b-row v-if="error">
        <b-col class="col-md-6 mx-auto">
          <b-alert variant="danger" :show= 3 dismissible @dismissed = "error = false"><strong>Uh oh! Check your spelling or spacing</strong></b-alert>
        </b-col>
      </b-row>
      <b-row v-if="empty">
        <b-col class="col-md-6 mx-auto">
          <b-alert variant="warning" :show= 3 dismissible @dismissed = "empty = false"><strong>Come on!</strong> You gotta put something...</b-alert>
        </b-col>
      </b-row>
      <b-row v-if="results" class="mt-3">
        <b-col>
          <!-- Main search results table -->
          <b-table
          outlined
          stacked = "md"
          hover
          :items = items
          :fields = fields
          >
          <template slot="more details" slot-scope="row">
            <font-awesome-icon icon="chevron-circle-down" @click.stop="row.toggleDetails"></font-awesome-icon>
          </template>
          <!-- Shows detailed temperature and pressure data -->
          <template slot="row-details" slot-scope="row">
            <b-card>
              <b-row>
                <b-col>
                  <b-table
                  hover
                  stacked = "md"
                  :items = [row.item.data.main]
                  >
                  </b-table>
                </b-col>
              </b-row>
            </b-card>
          </template>
          </b-table>
        </b-col>
      </b-row>
      <b-row class="mx-auto">
        <b-col>
          <b-button size="sm" @click="showRecentResults()">{{ showRecent ? 'Hide' : 'Show' }}  Recent Searches</b-button>
        </b-col>
      </b-row>
      <b-row class="mt-5" v-if="showRecent">
        <b-col>
            <b-table
              hover
              stacked = "md"
              show-empty
              :items = recentSearches
              :fields = fields >
              <template slot="more details" slot-scope="row">
                <font-awesome-icon icon="chevron-circle-down" @click.stop="row.toggleDetails"></font-awesome-icon>
              </template>
              <!-- Shows detailed temperature and pressure data -->
              <template slot="row-details" slot-scope="row">
                <b-card>
                  <b-row>
                    <b-col>
                      <b-table
                      stacked = "md"
                      :items = [row.item.data.main]
                      >
                      </b-table>
                    </b-col>
                  </b-row>
                </b-card>
              </template>
            </b-table>
        </b-col>
      </b-row>
    </div>
  </div>
</template>
<script>
import axios from 'axios';

export default {
  name: 'app',
  data () {
    return {
      empty: false,
      error:false,
      fields:[
        'city',
        'country',
        { 
          'key':'current temp',
          'sortable': true
        },
        'sky conditions',
        'more details'
      ],
      
      items:[],
      search: "",
      selected:"",
      showRecent:false,
      recentSearches:[],
      results: false,
    }
  },
  computed: {
    oneSpace: function(){
      if (this.search.includes("  ")){
        this.error = true;
      }
    }
  },
  methods: {
    searchCity: function(){
      this.oneSpace;
      if (this.search){
        let unitsOfMeasure;
        if (this.selected){
          unitsOfMeasure = 'imperial';
        } else {
          unitsOfMeasure = 'metric';
        }
        axios
        .get('http://api.openweathermap.org/data/2.5/weather?q=' + this.search + '&units=' + unitsOfMeasure + '&APPID=416d51bbc53e459a19800cf970e29249')
        .then(response => {
          //parse through response for data I want
          let city = response.data.name;
          let country = response.data.sys.country;
          let currentTemp = Math.round(response.data.main.temp);
          let skyCond = response.data.weather[0].description
          this.results = true;
          this.items = [{'city': city, 'country': country, 'current temp': currentTemp, 'sky conditions': skyCond, 'data':response.data}];
          this.recentSearches.push({'city': city, 'country': country, 'current temp': currentTemp, 'sky conditions': skyCond, 'data':response.data});
        })
        .catch(error => {
          this.error = true;
        });
      } else {
        this.empty = true;
      }
    },
    showRecentResults: function(){
      this.showRecent = !this.showRecent;
    }
  }
}


</script>

<style>
#app {
  background: url('./assets/sky.jpg') no-repeat center center fixed; 
  -webkit-background-size: cover;
  -moz-background-size: cover;
  -o-background-size: cover;
  background-size: cover;
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  overflow: hidden;
  min-height: -webkit-fill-available;
}
h1 {
  font-weight: normal;
  font-family: 'Montserrat Alternates', sans-serif,'Avenir', Helvetica, Arial, sans-serif;
  font-size: 6vw;
}

form {
  margin-top: 7%;
}
th{
  font-weight: 900;
}
td {
  font-weight: 500;
}

</style>
