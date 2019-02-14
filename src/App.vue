<template>
  <div id="app">
    <div class="container">
      <b-row>
        <b-col class="mt-5">
          <h1>Let's get some weather!</h1>
        </b-col>
      </b-row>
      <b-form @submit.prevent="searchCity()">
        <b-row class="mt-4">
          <b-col class="col-6 mx-auto">
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
            <b-form-group label="Want fahrenheit because you're an American?" class="custom-control custom-checkbox ml-0 p-0">
              <!-- <b-form-checkbox-group id="checkGroup"> -->
                  <input
                    v-model="selected"
                    id="customCheck1"
                    type= "checkbox"
                  >
                <label for = "customCheck1">Yes</label>
              <!-- </b-form-checkbox-group> -->
            </b-form-group>
          </b-col>
        </b-row>
      </b-form>  
      <b-row v-if="error">
        <b-col class="col-6 mx-auto">
          <b-alert variant="danger" :show= 3 dismissible @dismissed = "error = false"><strong>Uh oh! Check your spelling or spacing</strong></b-alert>
        </b-col>
      </b-row>
      <b-row v-if="empty">
        <b-col class="col-6 mx-auto">
          <b-alert variant="warning" :show= 3 dismissible @dismissed = "empty = false"><strong>Come on!</strong> You gotta put something...</b-alert>
        </b-col>
      </b-row>
      <b-row v-if="results" class="mt-3">
        <b-col>
          <b-table
          stacked = "md"
          hover
          :items = items
          :fields = fields
          >
          <template slot="more details" slot-scope="row">
            <font-awesome-icon icon="chevron-circle-down" @click.stop="row.toggleDetails"></font-awesome-icon>
          </template>
          <template slot="row-details" slot-scope="row">
            <!-- Shows detailed temperature and pressure data -->
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
      <b-row class="mx-auto">
        <b-col>
          <b-button size="sm" @click="showRecentResults()">{{ showRecent ? 'Hide' : 'Show' }}  Recent Searches</b-button>
        </b-col>
      </b-row>
      <b-row class="mt-5" v-if="showRecent">
        <b-col>
            <b-table
              stacked = "md"
              show-empty
              :items = recentSearches
              :fields = fields >
              <template slot="more details" slot-scope="row">
                <font-awesome-icon icon="chevron-circle-down" @click.stop="row.toggleDetails"></font-awesome-icon>
              </template>
              <template slot="row-details" slot-scope="row">
                <!-- Shows detailed temperature and pressure data -->
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
          //city
          let city = response.data.name;
          //country
          let country = response.data.sys.country;
          //current temp
          let currentTemp = Math.round(response.data.main.temp);
          //sky conditions
          let skyCond = response.data.weather[0].description
          // console.log(response);
          this.results = true;
          this.items = [{'city': city, 'country': country, 'current temp': currentTemp, 'sky conditions': skyCond, 'data':response.data}];
          this.recentSearches.push({'city': city, 'country': country, 'current temp': currentTemp, 'sky conditions': skyCond, 'data':response.data});
          console.log(this.recentSearches)
        })
        .catch(error => {
          this.error = true;
          console.log(error);
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
// npm WARN babel-loader@6.4.1 requires a peer of webpack@1 || 2 || ^2.1.0-beta || ^2.2.0-rc but none is installed. You must install peer dependencies yourself.

</script>

<style>
* {
  
}
#app {
  background-image: url('./assets/sky.jpg');
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  min-height: -webkit-fill-available;
}

h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
@media only screen and (orientation: portrait) {
  #app{
    min-height: -webkit-fill-available;
  }
}
/* @media only screen and (orientation: landscape) {
  min-height: -webkit-fill-available;
} */

</style>
