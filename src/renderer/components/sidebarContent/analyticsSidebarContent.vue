<template>
  <div id="analytics-sidebar-content">
    <router-view></router-view>
    
    <div id="loader" v-if="spinner.loading">
      <scale-loader  :loading="spinner.loading" :color="spinner.color" :size="spinner.size" ></scale-loader>
    </div>


    <div id="conditional-render" v-else>
      
      
      <nav id="page-nav" class="navbar navbar-expand-lg navbar-light bg-light">
          <li class="navbar-brand">
          <h1 id="dashboard-label">ANALYTICS</h1>
        </li>
        <div id="navbar-icons">
        <ul class="navbar-nav mr-auto">
          <li class="nav-item">
            <span ><div id="user-initials"><strong>last modified:</strong> {{ this.allData.created.toDate().toLocaleString() }}</div></span>  
          </li>  
          <li class="nav-item">
              <svg id="notif" xmlns="http://www.w3.org/2000/svg" height="24" viewBox="0 0 24 24" width="24"><path d="M12 22c1.1 0 2-.9 2-2h-4c0 1.1.89 2 2 2zm6-6v-5c0-3.07-1.64-5.64-4.5-6.32V4c0-.83-.67-1.5-1.5-1.5s-1.5.67-1.5 1.5v.68C7.63 5.36 6 7.92 6 11v5l-2 2v1h16v-1l-2-2z"/></svg>
            </li>
          <li class="nav-item">
            <span class="dot"><div id="user-initials">{{ storeState.user["first-name"].charAt(0) }}{{ storeState.user["surname"].charAt(0)}}</div></span>  
          </li>   
           
          <li class="nav-item">
            <!-- <span class="dot"><div id="user-initials">{{ storeState.user["first-name"].charAt(0) }}{{ storeState.user["surname"].charAt(0)}}</div></span>   -->
            <div @click="reload()" id="reload">
              <svg width="3em" height="2.5em" viewBox="0 0 16 16" class="bi bi-arrow-counterclockwise" fill="currentColor" xmlns="http://www.w3.org/2000/svg">
              <path fill-rule="evenodd" d="M12.83 6.706a5 5 0 0 0-7.103-3.16.5.5 0 1 1-.454-.892A6 6 0 1 1 2.545 5.5a.5.5 0 1 1 .91.417 5 5 0 1 0 9.375.789z"/>
              <path fill-rule="evenodd" d="M7.854.146a.5.5 0 0 0-.708 0l-2.5 2.5a.5.5 0 0 0 0 .708l2.5 2.5a.5.5 0 1 0 .708-.708L5.707 3 7.854.854a.5.5 0 0 0 0-.708z"/>
            </svg>
            </div>

          </li> 
        </ul>
        </div>
      </nav>
  
  
    <div id= "charts">
  
      <div id="bar">
        <bar-chart :chart-data="chartdata" :options="options"></bar-chart>
      </div>
  
      <div id="cluster">
  
        <scatter-chart :chart-data="clusterData" :options="clusterChartOptions"></scatter-chart>
  
      </div>
  
      <div id="crime_in_area">
        <bar-chart :chart-data="crimeLocationStackedBar" :options="optionsCrimeLocationStackedBar"></bar-chart>
      </div>
  
      <div id="crime_in_area">
        <bar-chart :chart-data="prioritiesData" :options="optionsPrioritiesData"></bar-chart>
      </div>
    </div>

  </div>

    

  </div>
</template>

<script>
import {store} from "../../store/store"
import {db} from '../../../../static/js/fire_config'
import BarChart from "../charts/BarChart.js";
import ScatterChart from "../charts/ScatterChart";
import ScaleLoader from 'vue-spinner/src/ScaleLoader'

import firebase from "firebase/app";
import "firebase/firestore";


/*
'#4dc9f6','#f67019','#f53794','#537bc4','#acc236','#166a8f','#00a950','#58595b','#8549ba' 
    */
export default {
  components: {
      BarChart,
      ScatterChart,
      ScaleLoader
    },
  data () {
    return {
      colors: ["#F44336","#9C27B0","#673AB7","#2196F3","#00BCD4","#FFEB3B","#FF5722","#607D8B",'#4dc9f6','#f67019','#f53794','#537bc4','#acc236','#166a8f','#00a950','#58595b','#8549ba',"#8d3025","#df154f","#a5b4cf","#13e8c6","#f27138","#894552","#e7d373","#10fc66","#b48dd3","#c7d0da","#7bc2ea","#725969","#055911" ],
      options:{},
      chartdata: {},
      storeState: store.state,
      allData: {},
      clusterData:{},
      clusterChartOptions:{},
      crimeLocationStackedBar: {},
      optionsCrimeLocationStackedBar: {},
      priorities: {},
      prioritiesData:{},
      optionsPrioritiesData:{},
      spinner:{
        loading: true,
        size: "200px",
        color: "#9FA8DA"
        },

    }
  },
  created(){

    const usersRef = db.collection("cached").doc("analytics")


    

      usersRef.get()
        .then((docSnapshot) => {
          if (docSnapshot.exists) {
            usersRef.onSnapshot((doc) => {
              /////////
              console.log("from cache")
              // console.log(doc.data())
              // console.log(doc)
              
              this.allData = doc.data();
              console.log(`created: ${typeof(this.allData.created)}||| ${this.allData.created.toDate()}  ||| ${this.allData.created}`)
              this.fillDataOverallCounts()
              this.fillDataCluster()
              this.fillCrimeLocationStackedBar();


                  fetch('http://localhost:8081/allPriority', {
                  method: 'GET',
                  mode: 'cors'
                  }).
                  then(response => response.json())
                  .then(({priorities}) => {
                    // console.log('Success2:', priorities);
                    this.priorities = priorities;
                  }).then(()=>{
                    this.fillPrioritiesData()
                  })

              //////////////
            })
          } else {

                    
            fetch('http://localhost:8081/CrimeAnalytics?orientCluster=records', {
              method: 'GET',
              mode: 'cors'
            })
            .then(response => response.json())
            .then(data => {
              // console.log('Success:', data);
              this.allData = data;
              // console.log("new")
              // console.log({...data,created: new Date()})
              this.cacheTofirebase({...data,created: new Date()/* firebase.firestore.FieldValue.serverTimestamp */});
            }).then(()=>{
              this.fillDataOverallCounts()
              this.fillDataCluster()
              this.fillCrimeLocationStackedBar();


                  fetch('http://localhost:8081/allPriority', {
                  method: 'GET',
                  mode: 'cors'
                  }).
                  then(response => response.json())
                  .then(({priorities}) => {
                    // console.log('Success2:', priorities);
                    this.priorities = priorities;
                  }).then(()=>{
                    this.fillPrioritiesData()
                  })
            }).then(()=>{
              this.spinner.loading = false;
            })
            .catch((error) => {
              console.error('Error:', error);
            });



          }
      }).then(()=>{
        this.spinner.loading = false;
      });

    // fetch('http://localhost:8081/CrimeAnalytics?orientCluster=records', {
    //   method: 'GET',
    //   mode: 'cors'
    // })
    // .then(response => response.json())
    // .then(data => {
    //   // console.log('Success:', data);
    //   this.allData = data;
    //   this.cacheTofirebase(data);
    // }).then(()=>{
    //   this.fillDataOverallCounts()
    //   this.fillDataCluster()
    //   this.fillCrimeLocationStackedBar();


    //       fetch('http://localhost:8081/allPriority', {
    //       method: 'GET',
    //       mode: 'cors'
    //       }).
    //       then(response => response.json())
    //       .then(({priorities}) => {
    //         // console.log('Success2:', priorities);
    //         this.priorities = priorities;
    //       }).then(()=>{
    //         this.fillPrioritiesData()
    //       })
    // }).then(()=>{
    //   this.spinner.loading = false;
    // })
    // .catch((error) => {
    //   console.error('Error:', error);
    // });

  },
  methods:{
   fillDataOverallCounts(){
    
    const {over_counts} = this.allData;
    let labs = Object.keys(over_counts);
    this.chartdata = {
      labels: labs,
      datasets: [
        {
          label: 'Overall Crime Counts',
          backgroundColor: '#f87979',
          data: Object.values(over_counts)
        }
      ]
    }
    this.options = {
      responsive: true,
      maintainAspectRatio: false,
      title:{
        display: true,
        text: "Overall Crime Counts",
        fontSize: 25
      }
    }
    // console.log(this.scatter)
    
  },
  fillDataCluster(){
   
    //Theft	Embezzelment
    const x_axis = "Theft"
    const y_axis = "Embezzelment"
    let store = Object.entries(this.xyScatter(x_axis,y_axis))
   
    let datasets = []
    let tempcolor = [...this.colors]
    store.forEach((el)=>{
      let col = tempcolor.pop()
      let obj = {
        label: ""+el[0],
        data: el[1],
        pointBackgroundColor: col,
        backgroundColor: col
      }
      datasets.push(obj)
    })

    this.clusterData= {
      
      datasets
    }
    
    

    this.clusterChartOptions = {
      responsive: true,
      maintainAspectRatio: false,
      title:{
        display: true,
        text: `Cluster Data ${x_axis} vs ${y_axis}`,
        fontSize: 25
      },
      scales: {
              yAxes: [{
                scaleLabel: {
                  display: true,
                  labelString: y_axis
                }
              }],
              xAxes: [{
                scaleLabel: {
                  display: true,
                  labelString: x_axis
                }
              }]
        }    
    }
  },
  xyScatter(xName,yName){

    const {"overall_tables":clustersTable} = this.allData;
    
    let clusters = {}

    clustersTable.forEach((el)=>{

      const {"Crime_clusters":numCluster} = el;
      if (numCluster in clusters == false) clusters[numCluster] = [];
    })
    

    clustersTable.forEach((el)=>{
      
      let {"Crime_clusters":numCluster,[xName]:x,[yName]:y} = el;
      clusters[numCluster].push({x,y})

    })

    return clusters

  },
  fillCrimeLocationStackedBar(){
    const {"overall_tables":clustersTable,offences,locations} = this.allData;

    const location = {}
    const store = []

    

        for (const off of offences) {
          const temp = []
          clustersTable.forEach((e)=>{
            
            
            const {[off]:offval} = e;
            temp.push(offval);

          })
          store.push(temp)
          
        }


    let datasets = []
    let tempcolor = [...this.colors]
    store.forEach((el,i)=>{
      
      let col = tempcolor.pop()
      let obj = {
        label: ""+offences[i],
        data: el,
        pointBackgroundColor: col,
        backgroundColor: col
      }
      datasets.push(obj)
    })    

    this.crimeLocationStackedBar = {
      labels: locations,
      datasets
    }

    this.optionsCrimeLocationStackedBar = {
      title: {
						display: true,
						text: 'Crimes by location'
					},
					tooltips: {
						mode: 'index',
						intersect: false
					},
					responsive: true,
					/* scales: {
						x: {
							stacked: true,
						},
						y: {
							stacked: true
						}
          } */
          scales: {
              xAxes: [{
                stacked: true,
                gridLines: {
                  display: false,
                }
              }],
              yAxes: [{
                stacked: true,
                ticks: {
                  beginAtZero: true,
                },
                type: 'linear',
              }]
            }
    }

  },
  fillPrioritiesData(){
    const labs = Object.keys(this.priorities)
    const data = Object.values(this.priorities);

    this.prioritiesData = {
      labels: labs,
      datasets: [
        {
          label: "Priorities",
          backgroundColor: this.colors[6],
          data: data
        }
      ]
    }
    this.optionsPrioritiesData = {
      responsive: true,
      maintainAspectRatio: false,
      title:{
        display: true,
        text: "Priorities",
        fontSize: 25
      }
    }
  },
  cacheTofirebase(data){
    // Add a new document in collection "cities"
    db.collection("cached").doc("analytics").set(data)
    .then(function() {
        console.log("Document successfully written!");
    })
    .catch(function(error) {
        console.error("Error writing document: ", error);
    });
  },
  collectGraphData(){
    fetch('http://localhost:8081/CrimeAnalytics?orientCluster=records', {
              method: 'GET',
              mode: 'cors'
            })
            .then(response => response.json())
            .then(data => {
              // console.log('Success:', data);
              this.allData = data;
              this.cacheTofirebase({...data,created: firebase.firestore.Timestamp.fromDate(new Date())});
              console.log("reload")
              console.log(firebase.firestore.Timestamp.fromDate(new Date()))

            }).then(()=>{
              this.fillDataOverallCounts()
              this.fillDataCluster()
              this.fillCrimeLocationStackedBar();


                  fetch('http://localhost:8081/allPriority', {
                  method: 'GET',
                  mode: 'cors'
                  }).
                  then(response => response.json())
                  .then(({priorities}) => {
                    // console.log('Success2:', priorities);
                    this.priorities = priorities;
                  }).then(()=>{
                    this.fillPrioritiesData()
                  })
            }).then(()=>{
              this.spinner.loading = false;
            })
            .catch((error) => {
              console.error('Error:', error);
            });
  },
  reload(){

    this.spinner.loading = true;
    this.collectGraphData()

  }

  }
}
</script>

<style>
#analytics-sidebar-content {
  background-color: #F8F9F9;
  width: 100%;
  height: 100%;
  margin-left: 230px
}

#charts{
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
}

#charts div{
  width: 100%;
  max-width: 600px;
  /* min-width: 600px; */
  height: auto;
  
}
#loader{
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh
}
#reload svg{
  margin-top: 5px;
}

#reload{
  
  margin-left: 20px;
}
</style>