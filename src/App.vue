<template>
  <div id="app_vue" >


    <div id="nav">
      <router-link    tag='button'                    class="navButn"  to="/"      >Home</router-link>
      <router-link v-if="showLogin"   tag='button'    class="navButn"  to="/login" >Login</router-link>
      <router-link v-if="showLogin"   tag='button'    class="navButn"  to="/signUp">Sign Up</router-link>
      <router-link v-if="showData"    tag='button'    class="navButn"  to="/Data"  >Data</router-link>
      <button      v-if="showData"    @click="logOut"                       class="navButn"              >logOut</button>
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
        status: "",
        showLogin: true,
        showData: false
      }
    },mounted(){
      this.checkSession()
    },
    methods:{
      async logOut  () {
        localStorage.session = false;
        this.$router.push({name: 'home'})
        console.log(localStorage.session);
        console.log("se eejcuta");
        this.checkSession()
      },
      async getStatus  () {
        console.log("la sessio esta en");
        console.log(localStorage.session);
        this.status = localStorage.session;
      },
      async checkSession() {
        if (localStorage.session) {
          if (localStorage.session == "true") {
              this.showLogin = false;
              this.showData = true;
          }else{
            this.showLogin = true
            this.showData = false
          }
      }
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


#nav a.router-link-exact-active {
  color: #694ed6;
}
</style>
