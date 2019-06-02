<template>
    <div  class="login">
        <h1>welcome to data {{name}}</h1>
        <h2>{{email}}</h2>
        <h2>{{token}}</h2>
        <h3>{{number}}</h3>


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
        <div class="createCard" v-if="showCreateFolder" >
            <div class="cardContent" >
                <div class="cardTitle" >
                    Crear carpeta
                </div>
                <div >
                    <input v-model="folderName" type="text" placeholder="Nombre de la carpeta">
                </div>
                <div style="float:left;margin-left:80px">
                    <h3>{{advise}}</h3>
                    <button class="btn" @click="showCreateFolderFN">close create folder </button>
                    <button class="btn" @click="createFolder">create folder </button>
                </div>

            </div>
        </div>


        <button class="btn" @click="getChampions">get champions </button>
        <button class="btn" @click="deleteFile">delete file </button>
        <button class="btn" @click="getList">get List </button>
        <button class="btn" @click="showCreateFolderFN">create folder </button>
        <h1>files: {{files}}</h1>
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
                upload: [],
                showCreateFolder: false,
                folderName: "",
                number: 1,
                advise:"",
                files:[]
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

            async deleteFile  () {
                this.number +=1;
            },
            async showCreateFolderFN  () {
                this.showCreateFolder = !this.showCreateFolder;
            },
            async getList(){
                const res = await axios.post('http://192.168.99.101:5000/graphql', {
                        query:`mutation{
                                downloadList(owner:{
                                    owner:"sergio"
                                },input:{
                                    email:"`+localStorage.email+`",
                                    token:"`+localStorage.token+`"
                                }){
                                    name
                                    owner
                                    path
                                    ...filesRecursive
                                }
                                }
                                fragment filesRecursive on File {
                                files{
                                    ...fileFields
                                    files{
                                        ...fileFields
                                    files{
                                        ...fileFields
                                        files{
                                            ...fileFields
                                            files{
                                                ...fileFields
                                            }
                                        }
                                    }
                                    }
                                }
                                }

                                fragment fileFields on File {
                                    name
                                    owner
                                    path
                                }`
                })
                console.log(res.data.data);
                this.files =res.data.data.downloadList
            }
            ,
            async createFolder  () {
                if(this.folderName != ""){

                    const res = await axios.post('http://192.168.99.101:5000/graphql', {
                        query:`mutation{
                            createFolder(create:{
                                owner: "`+localStorage.name+`"
                            path: "/SharedStorage/`+this.folderName+`"
                        },input:{
                            email: "`+localStorage.email+`"
                            token: "`+localStorage.token+`"
                        }){
                            create_id
                            owner
                            path
                            date
                            advise
                        }
                    }`
                })
                console.log(res.data.data.createFolder);
                }else{
                    this.advise = "El nombre no puede estar vacio"
                    setTimeout(() => this.advise="", 3000);
                }
            },
            async getChampions(){
                const res = await axios.post('http://localhost:4000/graphql',{
                    query: "{ getChampions{name attackDamage link_page}}"
                })
                this.champions = res.data.data
            },
            async processFile(event) {
                let data= new FormData()
                let file =  event.target.files[0]
                //console.log(file.name);
                let operations ='{ "query": "mutation($uploads: [Upload!]!){  uploadFiles(files:{    uploads:$uploads    name: \\"'+file.name+'\\"    description:\\"prueba gql\\"    owner: \\"'+localStorage.name+'\\"  },input:{email:\\"'+localStorage.email+'\\" token: \\"'+localStorage.token+'\\"})  {    name description owner path advise  }}", "variables": { "uploads": [null] } }'
                data.append("operations",operations)
                data.append('map','{ "0": ["variables.uploads.0"] }')
                data.append('0',file)

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


    curl http://192.168.99.101:5000/graphql   -F operations='{ "query": "mutation($uploads: [Upload!]!){\n  uploadFiles(files:{\n    uploads:$uploads\n    name: \"graphql\"\n    description:\"prueba gql\"\n    owner: \"david\"\n  },input:{email:\"dsgantivac@unal.edu.co\"\n token: \"BdDuQjgGnW\"})\n  {\n    name\n description\n owner\n path\n advise\n  }\n}\n", "variables": { "uploads": [null] } }'   -F map='{ "0": ["variables.uploads.0"] }'   -F 0=@/home/gantiva/Downloads/logoFinal.jpg





-->