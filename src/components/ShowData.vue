<template>
    <div  class="login">
        <h1>welcome to data {{name}}</h1>
        <h2>{{email}}</h2>
        <h2>{{token}}</h2>


        <div v-for="(elements,index) in champions" :key="index">
            <div v-for="(champion,index) in elements" :key="index">
                <a v-bind:href="champion.link_page">
                <div class="folder-container">
                    <img height="120px" src="../assets/folder3.png" >
                    <div class="centered">{{champion.name}} </div>
                </div>
                </a>
            </div>
        </div>
        <h1>upload: {{upload}}</h1>
        <div style="clear:both;"></div>

        <div class="upload" >
            <label  for="file" style="cursor:pointer">
                upload file  <span class="glyphicon glyphicon-upload"></span>
            </label>
            <input class="inputfile" id="file" type="file" @change="processFile($event)">
        </div>
        <button class="btn" @click="getChampions">Get champions</button>
    </div>
</template>

<script>
    import axios from 'axios'
    export default {
        name: 'ShowData',
        data(){
            return{
                champions: [],
                name:"",
                email:"dsgantivac@unal.edu.co",
                password:"12345678",
                token:"",
                someData: [],
                upload: []
            }
        },
        mounted(){
            if (localStorage.session) {
                if (localStorage.session != "true") {
                    this.is_signed = localStorage.session;
                    this.$router.push({name: 'login'})
                }else{
                    this.name = localStorage.name
                    this.email = localStorage.email
                    this.token = localStorage.token
                    console.log("in session");
                }
            }
        }
        ,methods: {

            async getChampions(){
                const res = await axios.post('http://localhost:4000/graphql',{
                    query: "{ getChampions{name attackDamage link_page}}"
                })
                this.champions = res.data.data
            },
            async processFile(event) {
                let data= new FormData()
                let file =  event.target.files[0]
                let operations = '"query": "mutation($uploads: [Upload!]!){\\n  uploadFiles(files:{\\n    uploads:$uploads\\n    name: \\"graphql\\"\\n    description:\\"prueba gql\\"\\n    owner: \\"diegun\\"\\n  },input:{email:\\"'+localStorage.email+'\\"\\n token: \\"'+localStorage.token+'\\"})\\n  {\\n    name\\n description\\n owner\\n path\\n advise\\n  }\\n}\\n", "variables": { "uploads": [null]}'
                data.append("operations",operations)
                data.append('map',file)
                data.append('file',file)

                const res = await axios.post('http://192.168.99.101:5000/graphql',data,{
                    headers: { 'Content-Type': 'multipart/form-data'  }
                } )

                console.log(file);
                this.upload = res.data.data
            }

        },
    }

</script>

<style scoped src='../styles/Data.css' >

</style>

<!--
https://stackoverflow.com/questions/49587831/how-to-append-files-and-data-to-formdata-in-vue-js

var files=[fs.createReadStream('/home/diego/Imágenes/test.png')]
    var options = {
      method: 'POST',
      url: 'http://localhost:5000/graphql',
      headers:
        { 'Content-Type': 'multipart/form-data' },
      formData:
      {
        operations: '{ "query": "mutation($uploads: [Upload!]!){\\n  uploadFiles(files:{\\n    uploads:$uploads\\n    name: \\"graphql\\"\\n    description:\\"prueba gql\\"\\n    owner: \\"diegun\\"\\n  },input:{email:\\"unemail2@gmail.com\\"\\n token: \\"QnIcrnWpvU\\"})\\n  {\\n    name\\n description\\n owner\\n path\\n advise\\n  }\\n}\\n", "variables": { "uploads": '+ files+'} }'
      }
    };
-->