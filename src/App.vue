<template>
  <div id="app">
    <b-container>
      <b-row>
        <b-col cols="4"> 
          <b-container>
            <b-row class="mb-2">  
              <b-form-select v-model="appName" :options="appNames" class="mb-3">
              </b-form-select>
            </b-row>
            <b-row class="mb-2">  
              <b-form-select v-model="endpoint" :options="endpointNames" class="mb-3">
              </b-form-select>
            </b-row>
            <b-row class="mb-2">  <b-button @click="onClick">View</b-button> </b-row>
          </b-container>
        </b-col>
        <b-col cols="8"> <VueJsonToTableVue :data.sync="jsonData" :endpoint="endpoint" :appName="appName"></VueJsonToTableVue> </b-col>
      </b-row>
    </b-container>
  </div>
</template>

<script>
import VueJsonToTableVue from './components/VueJsonToTable.vue'
import axios from 'axios'
import appEndpoints from '../config.json'

export default {
  name: 'App',
  components: {
    VueJsonToTableVue
  },
  data() {
    return {
      jsonData: JSON.parse('{"example": "table"}'),
      appNames: this.getOption(appEndpoints["Apps"], "Please select an application"),
      endpointNames: this.getOption(appEndpoints["Endpoints"], "Please select an endpoint"),
      appName: null,
      endpoint: null
    }
  },
  methods: {
    onClick: function() {
      axios
      .get(`http://${this.appName}/actuator/${this.endpoint}`) //`${this.appName}/actuator/${this.endpoint}`)
      .then((response) => {
        if (this.endpoint === 'metrics') {
          for (let i in response.data.names) {
            const metric = response.data.names[i];
            axios.
            get(`http://${this.appName}/actuator/${this.endpoint}/${metric}`)
            .then((res) => {
              if (res.data.measurements !== undefined && res.data.measurements.length > 0) {
                response.data.names[i] = response.data.names[i] + ` -- ${res.data.measurements[0].statistic}: ${res.data.measurements[0].value} ${res.data.baseUnit}`;
              }
              if (i == response.data.names.length - 1) {
                this.jsonData = response.data;
              }
            });
            
          }
        }
      });
    },
    getOption: function(jdata, defaultMessage) {
      let res = [{value: null, text: defaultMessage}];
      for (const da of jdata) {
        res.push({value: da, text: da});
      }
      return res;
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
