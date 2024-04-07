<template>
  <v-container fluid>
    <v-row>
      <v-col xs="12" md="4">
        <v-card class="dashboard-card" color="primary">
          <v-card-title class="text-center white--text title">
            Mother Patient <v-icon x-large color="white">mdi-human-female</v-icon>
          </v-card-title>
          <v-card-text class="text-center">
            <div class="headline white--text value mt-16"><h1 style="font-size:80px;">10</h1></div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col xs="12" md="4">
        <v-card class="dashboard-card" color="info">
          <v-card-title class="text-center white--text title">
            Baby Patient <v-icon x-large color="white">mdi-baby</v-icon>
          </v-card-title>
          <v-card-text class="text-center">
            <div class="headline white--text value mt-16"><h1 style="font-size:80px;">10</h1></div>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col xs="12" md="4">
        <v-card class="dashboard-card" color="success">
          <v-card-title class="text-center white--text title">
            New Born <v-icon x-large color="white">mdi-baby-buggy</v-icon>
          </v-card-title>
          <v-card-text class="text-center">
            <div class="headline white--text value mt-16"><h1 style="font-size:80px;">10</h1></div>
          </v-card-text>
        </v-card>
      </v-col>
    </v-row>
    <v-row>
      <v-col xs="12" md="6">
        <v-card>
          <v-card-title>Doctor List</v-card-title>
          <v-card-text class="mt-5">
            <v-responsive>
              <v-simple-table dense height="465">
                <thead>
                  <tr>
                    <th>Name</th>
                    <th>Position</th>
                  </tr>
                </thead>
                <tbody>
                  <tr v-for="(doctor, index) in doctorData" :key="index">
                    <td>{{ doctor.name }}</td>
                    <td>{{ doctor.position }}</td>
                  </tr>
                </tbody>
              </v-simple-table>
            </v-responsive>
          </v-card-text>
        </v-card>
      </v-col>
      <v-col xs="12" md="6">
        <v-card>
            <v-card-title>Chart</v-card-title>
            <v-card-text >
              <div class="small-chart-container">       
            <apexchart small type="bar" :options="chartOptions" :series="chartSeries"></apexchart>
              </div>
            </v-card-text>
        </v-card>
      </v-col>
    </v-row>
  </v-container>
</template>
<script>
import VueApexCharts from 'vue-apexcharts';
import { mapActions, mapState } from 'vuex'
export default {
    components: {
    apexchart: VueApexCharts,
  },
data(){
    return{
     headers:[
        {text:"Name",value:"name"},
        {text:"Position",value:"position"}
     ],

     //CHART
     chartOptions: {
        chart: {
          type: 'bar',
        },
        plotOptions: {
          bar: {
            horizontal: false,
          },
        },
        xaxis: {
          categories: ['Category 1', 'Category 2'],
        },
      },
      chartSeries: [
        {
          name: 'Series 1',
          data: [44, 55],
        },
      ],
    };
    
},
methods: {
        ...mapActions(['getDoctor']),
    
},
computed:{
        ...mapState([
            'rules',
            'doctorData',
        ]),
    },
mounted(){
        this.getDoctor()
    },
}
</script>

<style scoped>
.small-chart-container {
  width: 750px; /* Adjust the width as needed */
}
.dashboard-card {
  height: 250px;
}

.title {
  font-size: 24px; /* Adjust the title font size */
}

.value {
  font-size: 36px; /* Adjust the value font size */
}

/* Optional: Add more styles for responsiveness if needed */
</style>
