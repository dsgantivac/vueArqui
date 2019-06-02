<template>
  <div id="app_vue" >
    <!--
      <h1>status: {{status}}</h1>
    -->

    <div id="nav">
      <router-link tag='button' class="navButn" to="/">Home</router-link>
      <router-link tag='button' class="navButn" to="/login">Login</router-link>
      <router-link tag='button' class="navButn" to="/signUp">Sign Up</router-link>
      <router-link tag='button' class="navButn" to="/Data">Data</router-link>
      <button class="navButn" @click="logOut">logOut</button>
      <!--
        <button @click="getStatus">status</button>|
      -->
    </div>
    <router-view/>
  </div>
</template>

<script>
  export default {
    data(){
      return{
        status: ""
      }
    },
    methods:{
      async logOut  () {
        localStorage.session = false;
        this.$router.push({name: 'home'})
        console.log(localStorage.session);
        console.log("se eejcuta");
      },
      async getStatus  () {
        console.log("la sessio esta en");
        console.log(localStorage.session);
        this.status = localStorage.session;
      }

    }
  }


</script>


<style>

.navButn{
  border: none;
    background: #694ed6;
    cursor: pointer;
    border-radius: 15px;
    margin: 10px 10px 0px 10px;
    padding: 16px;
    width: 200px;
    color: white;
    box-shadow: 0 3px 6px 0 rgba(0, 0, 0, 0.2);
}


#app_vue {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}
#nav {
  padding: 30px;
}

#nav a {
  font-weight: bold;
  color: #2c3e50;
}

#nav a.router-link-exact-active {
  color: #694ed6;
}
</style>
