Page 1
# App.vue
<template>
  <div id="app">
    <Users/>
    

  </div>


</template>

<script>
import Users from './components/Users.vue'

export default {
  name: 'app',
  components: {
    Users
    
  }
 
}
</script>

# Users.vue

    <template>
  <div class="container">
    <h3>View Your Purchases</h3>
   
    
    <table class="table">
    
   
     
     
      <thead>
        <tr>
          <th scope="col">Id</th>
          <th scope="col">Asset</th>
          <th scope="col">Date/Time</th>
          <th scope="col">View</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="user in users" v-bind:key="user.id"> 
          <th scope="row">{{user.id}}</th>
          <td>{{user.name}}</td>
          <td>{{user.date/time}}</td>
         
          <td> <router-link><button type="button" onclick="document.location='index copy.html'">Details</button></router-link></td>
          
  
    
          
        </tr>
      </tbody>
    </table> 
  </div> 
 
       <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.3.1/css/bootstrap.min.css" integrity="sha384-ggOyR0iXCbMQv3Xipma34MD+dH/1fQ784/j6cY/iJTQUOhcWr7x9JvoRxT2MZw1T" crossorigin="anonymous">
    <title>tutorial</title>
  </head>
</template>
 
<script>
  import axios from 'axios';
  export default {
    name: 'Users',
    data() {
      return {
        users: null,
      };
    },
    created: function() {
      axios
        .get('https://jsonplaceholder.typicode.com/users')
        .then(res => {
          this.users = res.data;
        })
    }
  }
</script>
 
 
<style>
  h3 {
    margin-bottom: 5%;
  }

 
</style>
 
 
 
<style>

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #040505;
  margin-top: 60px;
}
</style>

#Index.html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app"></div>
    <!-- built files will be auto injected -->
  </body>
</html>



Page 2:
# Index copy.html
<!DOCTYPE html>
<html lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <link rel="icon" href="<%= BASE_URL %>favicon.ico">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <noscript>
      <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
    </noscript>
    <div id="app1"></div>
    <!-- built files will be auto injected -->
  </body>

  <h1>Asset Name</h1>
 
  <style>

    h1  {
      font-family: Avenir, Helvetica, Arial, sans-serif;
      -webkit-font-smoothing: antialiased;
      -moz-osx-font-smoothing: grayscale;
      text-align: center;
      color: #040505;
      margin-top: 50px;
    }
    </style>
<div>

 

  <head>
    <meta name="viewport" content="width=device-width, initial-scale=1">
   
    <style>
  
  .arrow {
      border: solid black;
      border-width: 0 3px 3px 0;
      display: inline-block;
      padding: 9px;
      margin-top: 30px;
      margin-left: 170px;
      position:absolute;
      transform: rotate(135deg);
      -webkit-transform: rotate(135deg);
      
    }
 
  
  </style>

</head>
    
    




<body>
  
  <router-link><a href='/Users' title="Return to home page"><div class="arrow " style="z-index:2"></div></a></router-link>
  

  <hr style="width:50%;text-align:center;margin-left:50">
  <router-link><a href='/Users' title="Return to home page"> <h4 style="margin-top: 30px; margin-left: 200px">
    Home
  </h4></a></router-link>
  
  <h3 style="margin-top: -40px; margin-left: 380px">Logistic company 1</h3>
  <h4 style="margin-top: 15px; margin-left: 380px">Asset ID:</h4>
  <h4 style="margin-top: 20px; margin-left: 380px">Car Licence Number:</h4>
  <h4 style="margin-top: 30px; margin-left: 380px"> Temperature:</h4>
  <h4 style="margin-top: 35px; margin-left: 380px">Time Stamp:</h4>
<style>

 
</style>

<hr style="width:50%;text-align:center;margin-left:50; margin-top:40px">
<h3 style="margin-top: 40px; margin-left: 380px">Pharmacy</h3>
<h4 style="margin-top: 40px; margin-left: 380px">Asset ID:</h4>
<h4 style="margin-top: 40px; margin-left: 380px">Car Licence Number:</h4>
<h4 style="margin-top: 40px; margin-left: 380px"> Temperature:</h4>
<h4 style="margin-top: 40px; margin-left: 380px">Time Stamp:</h4>
<hr style="width:50%;text-align:center;margin-left:50; margin-top:45px">
</body>

  </html>






    




    
   
   

  





    
