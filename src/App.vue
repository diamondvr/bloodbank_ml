<script>
import axios from 'axios'
import moment from 'moment'

export default {
  name: 'app',
  components: {
    
  },
  data() {
    return {
      server: "10.66.1.4:8081",
      token: "token ",
      dayToUpdate: null,
      record: null,
      detailList: []
    }
  },
  async mounted() {
    await this.getAuthentication()
    await axios.get("http://" + this.server + "/autoML/json/", 
    { "headers" : { "Authorization" : this.token }})
    .then(response => this.detailList = response.data.result)
    .catch(error => console.log(error));
    this.detailList.forEach((items, index) => 
      this.detailList[index].log_date = moment(items.log_date).format("DD MM YYYY")
    )
    await axios.get("http://" + this.server + "/autoML/time/", 
    { "headers" : { "Authorization" : this.token }})
    .then(response => {
      this.dayToUpdate = response.data.everyDays
      this.record = response.data.everyRecords
    })
    .catch(error => console.log(error));
  },
  methods: {
    getAuthentication: async function() {
      let data = JSON.stringify({
        password: "1234",
        username: "ivetwest"
      })
      await axios.post("http://" + this.server + "/api/v1/auth/login/", data,
      { "headers" : 
        { "Content-Type" : "application/json" }, 
      })
      .then(response => {
        this.token += response.data.result.key
      })
      .catch(error => { 
        console.log(error)
        alert("Unable to log in with provided credentials!") 
      })
    },
    isNumKey: function(event) {
      var charCode = (event.which) ? event.which : event.keyCode;
      if ((charCode > 31 && (charCode < 48 || charCode > 57)) && charCode !== 46) {
        event.preventDefault();
      } else {
        return true;
      }
    },
    saveSetting: async function() {
      if (this.dayToUpdate === '') {
        alert("The value can't be null!")
      } else {
        window.csrf_token = "{{ csrf_token() }}" 
        await axios.post("http://" + this.server + "/autoML/time/", 
        { "headers" : 
          { "Authorization" : this.token,
            "Content-Type" : "application/json" }, 
        }, JSON.stringify({"everyDays": this.dayToUpdate, "everyRecords": this.record}))
        .then(response => { 
          console.log(response)
          alert('Update setting successfully')
          return response  })
        .catch(error => { 
          console.log(error)
          alert("Error! Can't update setting") 
        })
      }
    },
    updateNow: async function() {
      if (this.record === '') {
        alert("The value can't be null!")
      } else {
        window.csrf_token = "{{ csrf_token() }}" 
        await axios.post("http://" + this.server + "/autoML/now/", 
        { "headers" : 
          { "Authorization" : this.token,
            "Content-Type" : "application/json" }, 
        }, JSON.stringify({"update": "1"}))
        .then(response => { 
          console.log(response)
          alert('Model Update is trigged!')
          return response  })
        .catch(error => { 
          console.log(error)
        })
      }
    }
  }
}
</script>

<template>
  <div id="app">
    <!-- <img alt="Vue logo" src="./assets/logo.png">
    <HelloWorld msg="Welcome to Your Vue.js App"/> -->
    <div class="header">
      <img alt="logo" src="./assets/logo.png" class="logo">
      <div class="title">
        <p>Blood Bank</p>
        <p>Machine Learning Configuration</p>
      </div>
    </div>
    <div class="body">
      <div class="formContainer">
        <form class="settingForm" onsubmit="return false">
          <p>Update every</p>
          <input type="text" v-model="dayToUpdate" @keypress="isNumKey($event)"> 
          <p>day after lastest update or </p>
          <p>update when data exceed</p>
          <input type="text" v-model="record" @keypress="isNumKey($event)"><br><br>
          <input type="submit" class="updateNowButt" value="Update" v-on:click="updateNow">
          <input type="submit" class="saveButt" value="Save" v-on:click="saveSetting">
        </form>
      </div>
      <div class="detailContainer">
        <p>Update History</p>
        <table class="detailTable">
          <thead>
            <tr>
              <th>Date</th>
              <th>Log Action</th>
              <th>Detail</th>
              <th>Update File</th>
          </tr>
          </thead>
          <tbody>
            <tr v-for="(ele, index) in detailList" v-bind:key="index">
              <td>{{ ele.log_date }}</td>
              <td>{{ ele.log_action }}</td>
              <td>{{ ele.log_detail }}</td>
              <td>{{ ele.csv_filename }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>
</template>

<style>
@import url('https://fonts.googleapis.com/css?family=Nunito:700&display=swap');
body {
   padding: 0;
   margin: 0;
   background-color: #2A3E50;
}

#app {
  font-family: 'Nunito', sans-serif;
}

.header {
  width: 100vw;
  display: flex;
  z-index: 1;
  background-color: #33495F;
  color: white;
  justify-content: center;
  align-items: center;
}

.title {
  display: inline-block;
  vertical-align: middle;
}

.title p {
  font-size: 28px;
  font-weight: bold;
  padding: 0;
  margin: 0 0 0 10px;
}

.logo {
  max-width: 85px;
  height: auto;
  display: inline-block;
  vertical-align: middle;
  margin-right: 5px;
}

.body {
  text-align: center;
  color: white;
}

.formContainer {
  margin: 30px auto;
  width: 70%;
  border-radius: 5px;
  background-color: #33495F;
  
}

.settingForm {
  padding: 15px;
}

.settingForm p {
  font-size: 20px;
  display: inline;
}

.settingForm input {
  margin: 0 10px 0 10px;
  font-size: 20px;
  width: 100px;
  text-align: center;
  border-radius: 3px;
}

.detailContainer {
  margin: 30px auto;
  width: 70%;
  border-radius: 5px;
  background-color: #33495F;
  
}

.detailContainer p {
  font-size: 22px;
  font-weight: bold;
  padding: 15px;
  padding-bottom: 0px; 

}

.detailTable {
  width: 90%;
  margin-left: auto;
  margin-right: auto;
  padding-bottom: 15px;
}

.detailTable thead {
  font-weight: bold;
  font-size: 20px;
  background-color: rgb(71, 71, 71);
}

.detailTable tbody {
  background-color: rgb(134, 134, 134);
}
</style>
