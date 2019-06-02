<template>
    <div  class="login">
        <h1>welcome to data {{name}}</h1>
        <h2 style="margin-bottom:30px;">{{email}}</h2>
        <h2 style="margin-bottom:30px;">{{token}}</h2>

        <div v-if="current_path.length > 1">
            <button class="btn" style="float:left" @click="popCurrentPath" >Retroceder</button>
        </div>
            <div style="clear:both;"></div>
            <div  v-for="(elements, index) in current_path[current_path.length-1]" :key="index" class="listContainer">
                <div class="folder-container" height="120px">
                    <div v-if="checkDot(elements.name)">
                        <div v-if="getExtension(elements.name) == 'txt' ">
                            <img height="120px" src="../assets/txt.png" >
                        </div>
                        <div v-else-if="getExtension(elements.name) == 'pdf' ">
                            <img height="120px" src="../assets/pdf.png" >
                        </div>
                        <div v-else>
                            <img height="120px" src="../assets/file.png" >
                        </div>
                        <div class="centered">{{addDot(elements.name)}} </div>
                    </div>

                    <div v-if="!checkDot(elements.name)">
                        <div @click="moveTo(elements.files)">
                            <img height="120px" src="../assets/folder3.png" >
                            <div class="centered"  >{{addDot(elements.name)}} </div>
                        </div>
                    </div>
                    <span @click="getOptions(elements.name,elements.path)"  class="glyphicon glyphicon-option-vertical bottom-right-image" ></span>
                </div>
            </div>

        <div style="clear:both;"></div>

        <div class="upload" >
            <label  for="file" style="cursor:pointer">
                upload file  <span class="glyphicon glyphicon-upload"></span>
            </label>
            <input class="inputfile" id="file" type="file" @change="processFile($event)">
        </div>

        <div class="OptionCard" v-if="showOptions">
            <div class="cardContent" style="height:330px" >
                <div class="cardTitle" >
                    Opciones de {{addDot(folderOptionName)}}
                </div>

                <h3>{{advise}}</h3>
                <div v-if="!MoveToShow" >
                    <button class="btn" @click="ShowMoveFile">Mover </button><br/>
                    <button class="btn" @click="hideOption">Eliminar </button><br/>
                    <button class="btn" @click="hideOption">Cancelar </button>
                </div>
                <!-- AQUI SE HACE EL MOVER LAS CARPETAS DE MONDA-->

                <div v-if="MoveToShow" >
                    <p>Selecciona la carpeta destino</p><br/>
                    <select v-model="fileDestiny">
                        <option v-if="current_path.length > 1 ">
                            Extraer de la carpeta
                        </option>
                        <option v-for="(item, index) in possiblePaths" :key="index">
                                {{item}}
                        </option>

                    </select>
                    <br/>
                    <button class="btn" @click="moveFile">Mover </button>
                    <button class="btn" @click="ShowMoveFile">Cancelar </button>
                </div>
            </div>
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
                    <button class="btn" @click="showCreateFolderFN">Cancelar </button>
                    <button class="btn" @click="createFolder">Crear carpeta </button>
                </div>
            </div>
        </div>


        <button class="btn" @click="deleteFile">delete file </button>
        <button class="btn" @click="showCreateFolderFN">create folder </button>
        <button class="btn" @click="moveFile">moveFile </button>
        <h1>Origin: {{fileOrigin}}</h1>
        <h1>Destiny: {{fileDestiny+addDot(folderOptionName)}}</h1>
        <h1>MoveReturn : {{moveReturn}}</h1>

    </div>
</template>

<script>
    import axios from 'axios';
    export default {
        name: 'ShowData',
        data(){
            return{
                name:"",
                email:"dsgantivac@unal.edu.co",
                password:"12345678",
                token:"",
                someData: [],
                upload: [],
                showCreateFolder: false,
                showOptions: false,
                folderOptionName: "",
                folderName: "",
                MoveToShow: false,
                number: 1,
                advise:"",
                user_files:[],
                current_path:[],
                moveReturn: [],
                possiblePaths: [],
                fileOrigin: "",
                fileDestiny: ""
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
                    this.getList()
                    this.current_path.push(this.user_files)
                }
            }
        },
        methods: {

            addDot: function (name) {
                return name.replace("_dot_",".")
            },
            checkDot: function(name){
                return(name.includes("_dot_"))
            },
            moveTo: function (fileList) {
                this.current_path.push(fileList)
            },
            popCurrentPath: function() {
                this.current_path.pop()
            },
            getExtension: function(filename){
                let ret = filename.split('_dot_').pop();
                return ret
            },
            getOptions: function(fName,path){
                this.showOptions = true
                this.folderOptionName = fName
                this.fileOrigin = path;
                console.log("displayed");
                console.log(this.fileOrigin);
            },
            hideOption: function () {
                this.showOptions = false
                this.folderOptionName = ""
                this.fileOrigin = ""
                this.fileDestiny = ""
            },
            adviseError: function(msg){
                this.advise = msg
                setTimeout(() => this.advise="", 3000);
            },
            ShowMoveFile: function () {
                let arr = this.current_path[this.current_path.length-1]
                this.possiblePaths = []
                for (let i = 0; i < arr.length; i++) {
                    if(!this.checkDot(arr[i].name)){
                        this.possiblePaths.push(arr[i].name)
                    }
                }
                console.log(this.possiblePaths);
                this.MoveToShow = !this.MoveToShow
            }
            ,
            async deleteFile  () {
                this.number +=1;
            },
            async showCreateFolderFN  () {
                this.showCreateFolder = !this.showCreateFolder;
            },
            async Download  () {
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
                let tmp = res.data.data.downloadList.files;

                for (let index = 0; index < tmp.length; index++) {
                    this.user_files.push(tmp[index])
                }
            }
            ,
            async createFolder  () {
                if(this.folderName != ""){

                    const res = await axios.post('http://192.168.99.101:5000/graphql', {
                        query:`mutation{
                            createFolder(create:{
                                owner: "sergio"
                            path: "/SharedStorage/sergio/`+this.folderName+`"
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
                console.log(res);
                }else{
                    this.adviseError("El nombre no puede estar vacio")
                }
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
            },
            async moveFile () {

                    if(this.fileDestiny == "Extraer de la carpeta"){
                        this.fileDestiny = this.fileOrigin.replace(this.addDot(this.folderOptionName),"")
                        this.fileDestiny = this.fileDestiny.replace("/sergio","")
                        this.fileDestiny = this.fileDestiny.replace("/SharedStorage","")
                        let tmp = this.fileDestiny.split("/")
                        this.fileDestiny = this.fileDestiny.replace(tmp[tmp.length - 2]+"/","")
                        console.log(this.fileDestiny);
                    }else{
                        this.fileDestiny = this.fileOrigin.replace(this.addDot(this.folderOptionName),"") + this.fileDestiny
                        this.fileDestiny = this.fileDestiny.replace("/sergio","")
                        this.fileDestiny = this.fileDestiny.replace("/SharedStorage","")
                        this.fileDestiny += "/"

                    }

                    const res = await axios.post('http://192.168.99.101:5000/graphql', {
                        query:`mutation{
                                    moveFile(move:{
                                        owner:"sergio"
                                        #origen, incluyendo el archivo (graphql en este caso)
                                        origin:"`+this.fileOrigin+`"
                                        #destino, incluyendo el archivo (graphql en este caso)
                                        destiny:"/SharedStorage/sergio/`+this.fileDestiny+this.addDot(this.folderOptionName)+`"

                                    },input:{
                                        email: "`+localStorage.email+`"
                                        token: "`+localStorage.token+`"
                                    })
                                    {
                                        move_id
                                        owner
                                        origin
                                        destiny
                                        date
                                        advise
                                    }
                                }
`
                })
                this.moveReturn = res.data.data.moveFile
                console.log(res.data.data);

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