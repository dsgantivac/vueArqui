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
                        <div @click="getDownload(elements.path)">
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
                    <button class="btn" @click="deleteFile">Eliminar </button><br/>
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
        <h1>Origin: {{fileOrigin+addDot(folderOptionName)}}</h1>
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
                fileDestiny: "",
//                hostname: "http://192.168.99.101:5000/graphql"
                hostname: "http://34.73.216.116:5000/graphql"
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
            async getDownload (filePath) {
                let tmpBase64 = "data:image/svg+xml;base64,PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0iVVRGLTgiIHN0YW5kYWxvbmU9Im5vIj8+CjxzdmcgeG1sbnM9Imh0dHA6Ly93d3cudzMub3JnLzIwMDAvc3ZnIiB4bWxuczp4bGluaz0iaHR0cDovL3d3dy53My5vcmcvMTk5OS94bGluayIgd2lkdGg9IjQwcHgiIGhlaWdodD0iNDBweCIgdmlld0JveD0iMCAwIDQwIDQwIiB2ZXJzaW9uPSIxLjEiPjx0aXRsZT5MYXVnaGluZzwvdGl0bGU+PGRlc2M+WW91ciB1bmlxdWUgdmFuaWxsaWNvbiBpcyBjcmFmdGVkIHdpdGggbG92ZSBieSB0aGUgdGVhbSBhdCB2YW5pbGxhZm9ydW1zLmNvbS48L2Rlc2M+PGRlZnMvPjxnIGlkPSJ2YW5pbGxpY29uIiBzdHJva2U9Im5vbmUiIHN0cm9rZS13aWR0aD0iMSIgZmlsbD0ibm9uZSIgZmlsbC1ydWxlPSJldmVub2RkIj48cmVjdCBpZD0iYmFja2dyb3VuZCIgZmlsbD0iI0Y0NjI0QyIgeD0iLTIiIHk9Ii0yIiB3aWR0aD0iNDQiIGhlaWdodD0iNDQiLz48ZyBpZD0iYXZhdGFyIiB0cmFuc2Zvcm09InJvdGF0ZSgtMTYsIDIwLCAyMCkgdHJhbnNsYXRlKDMuMDAwMDAwLCAzLjAwMDAwMCkiPjxjaXJjbGUgaWQ9ImJvZHkiIGZpbGw9IiNCNERFQzQiIGN4PSIxNyIgY3k9IjE3IiByPSIxNiIvPjxnIGlkPSJleWVzIiB0cmFuc2Zvcm09InRyYW5zbGF0ZSg5LjAwMDAwMCwgMTApIiBmaWxsPSIjNUM1QTY0Ij48cGF0aCBkPSJNMi41LDUgQzMuODgwNzExODcsNSA1LDMuODgwNzExODcgNSwyLjUgQzUsMS4xMTkyODgxMyAzLjg4MDcxMTg3LDAgMi41LDAgQzEuMTE5Mjg4MTMsMCAwLDEuMTE5Mjg4MTMgMCwyLjUgQzAsMy44ODA3MTE4NyAxLjExOTI4ODEzLDUgMi41LDUgWiIgaWQ9ImxlZnRfZXllIi8+PGNpcmNsZSBpZD0icmlnaHRfZXllIiBjeD0iMTMuNSIgY3k9IjIuNSIgcj0iMi41Ii8+PC9nPjxnIGlkPSJtb3V0aCIgdHJhbnNmb3JtPSJ0cmFuc2xhdGUoNi4wMDAwMDAsIDE4LjAwMDAwMCkiIGZpbGw9IiM1QzVBNjQiPjxwYXRoIGQ9Ik0yMC43OTk5NjM4LDAgTDEuMjAwMDM2MjMsMCBDMC41Njc4MTczNjcsMCAwLjA5NDI1NDI5MDQsMC41NzkzNDkzMjQgMC4yMjAwMjI5NzQsMS4xOTg5MzIxOSBDMS4yNTI5MjQ3Niw2LjI4NzM4NjkgNS43NDQ2OTcyNCwxMCAxMSwxMCBDMTYuMjU1MzAyOCwxMCAyMC43NDcwNzUyLDYuMjg3Mzg2OSAyMS43Nzk5NzcsMS4xOTg5MzIxOSBDMjEuOTA1NzQ1NywwLjU3OTM0OTMyNCAyMS40MzIxODI2LDAgMjAuNzk5OTYzOCwwIFoiIGlkPSJtb3V0aCIvPjwvZz48cGF0aCBkPSJNMzQsMTcgQzM0LDcuNjExMTU5MjUgMjYuMzg4ODQwNywwIDE3LDAgQzcuNjExMTU5MjUsMCAwLDcuNjExMTU5MjUgMCwxNyBDMCwyNi4zODg4NDA3IDcuNjExMTU5MjUsMzQgMTcsMzQgQzI2LjM4ODg0MDcsMzQgMzQsMjYuMzg4ODQwNyAzNCwxNyBaIE0zLDE3IEMzLDkuMjY4MDEzNSA5LjI2ODAxMzUsMyAxNywzIEMyNC43MzE5ODY1LDMgMzEsOS4yNjgwMTM1IDMxLDE3IEMzMSwyNC43MzE5ODY1IDI0LjczMTk4NjUsMzEgMTcsMzEgQzkuMjY4MDEzNSwzMSAzLDI0LjczMTk4NjUgMywxNyBaIiBpZD0iY2lyY2xlIiBmaWxsPSIjNUM1QTY0Ii8+PC9nPjwvZz48L3N2Zz4K"
                console.log("download mi perro");
                console.log(filePath);
            }
            ,
            async deleteFile  () {
                /**

                if(true){
                    if(this.checkDot(this.folderOptionName)){
                        console.log("es archivo");

                    }else{
                        console.log("Es carpeta");
                        this.fileOrigin = "/SharedStorage/sergio/"+this.folderOptionName
                    }
                    console.log(this.fileOrigin);

                }
       */

                    const res = await axios.post(this.hostname, {
                        query:`mutation{
                            deleteFile(del:{
                                owner: "prueba"
                                    path: "`+this.fileOrigin+`"
                                },input:{
                                    email: "`+localStorage.email+`"
                                    token: "`+localStorage.token+`"
                                }){
                                    del_id
                                    owner
                                    path
                                    date
                                    advise
                                }
                                }`
                })
                console.log(res.data.data);


            },
            async showCreateFolderFN  () {
                this.showCreateFolder = !this.showCreateFolder;
            },
            async Download  () {
                this.showCreateFolder = !this.showCreateFolder;
            },
            async getList(){
                console.log("se ejecuta el list");
                const res = await axios.post(this.hostname, {
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
                console.log(res);
                for (let index = 0; index < tmp.length; index++) {
                    this.user_files.push(tmp[index])
                }
            }
            ,
            async createFolder  () {
                if(this.folderName != ""){

                    const res = await axios.post(this.hostname, {
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
                let operations ='{ "query": "mutation($uploads: [Upload!]!){  uploadFiles(files:{    uploads:$uploads    name: \\"'+file.name+'\\"    description:\\"prueba gql\\"    owner: \\"sergio\\"  },input:{email:\\"'+localStorage.email+'\\" token: \\"'+localStorage.token+'\\"})  {    name description owner path advise  }}", "variables": { "uploads": [null] } }'
                data.append("operations",operations)
                data.append('map','{ "0": ["variables.uploads.0"] }')
                data.append('0',file)

                const res = await axios.post(this.hostname,data,{
                    headers: { 'Content-Type': 'multipart/form-data'  }
                } )

                console.log(file);
                this.upload = res.data.data
            },
            async moveFile () {

                    if(this.fileDestiny == "Extraer de la carpeta"){
                        console.log(this.fileOrigin);
                        this.fileOrigin = this.fileOrigin.replace("/SharedStorage","")
                        this.fileOrigin = this.fileOrigin.replace("/sergio","")
                        console.log(this.fileOrigin);

                        let tmp = this.fileOrigin.split("/")
                        //console.log(tmp);
                        this.fileDestiny = this.fileOrigin.replace("/"+tmp[tmp.length - 2],"")
                        console.log(this.fileDestiny);

                        //return;
                    }else{
                        if( this.fileOrigin == null){
                             this.fileOrigin = "/"+this.addDot(this.folderOptionName)
                        }
                        this.fileDestiny = this.fileOrigin.replace(this.addDot(this.folderOptionName),"") + this.fileDestiny
                        this.fileDestiny = this.fileDestiny.replace("/"+"sergio","")
                        this.fileDestiny = this.fileDestiny.replace("/SharedStorage","")
                        this.fileOrigin = this.fileOrigin.replace("/"+"sergio","")
                        this.fileOrigin = this.fileOrigin.replace("/SharedStorage","")
                        this.fileDestiny += "/"+this.addDot(this.folderOptionName)
                        console.log(this.fileOrigin);
                        console.log(this.fileDestiny);

                    }

                    const res = await axios.post(this.hostname, {
                        query:`mutation{
                                    moveFile(move:{
                                        owner:"sergio"
                                        #origen, incluyendo el archivo (graphql en este caso)
                                        origin:"/SharedStorage/sergio/`+this.fileOrigin+`"
                                        #destino, incluyendo el archivo (graphql en este caso)
                                        destiny:"/SharedStorage/sergio/`+this.fileDestiny+`"

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
                                }`
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