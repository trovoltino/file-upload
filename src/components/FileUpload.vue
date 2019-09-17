<template>
  <div class="file-upload">
    <div v-if="processingFile" class="cover"></div>
    <div id="file-drag-drop">
      <form ref="fileform" class="drop-form">
        <input v-model="email" v-bind:class="{ missingEmail: emailProvided }" type="text" placeholder="Please enter your Email to receve results" class="drop-email">
        <img v-if="!fileSent" v-bind:class="{ invisible: dropIconInvisible }" src="@/assets/images/download.svg" class="drop-icon" alt="drag and drop">
        <img v-if="fileSent" v-bind:class="{ active: fileSent }" src="@/assets/images/checked.svg" alt="">
        <span v-if="!fileSent" v-bind:class="{ invisible: !supportedFileFormat }" class="drop-files" name="sampleFile"><b>Choose PDF files</b> and drag it here.</span>
        <span v-else class="green"><b>File has been uploaded successfully</b>, we will send results shortly.</span>
        <label v-if="!fileSent" class="file-select">
          <div class="select-button">Select File</div>
          <input  type="file" id="file" ref="file" multiple v-on:change="handleFileUpload()"/>
        </label>
        <button v-else class="reset-btn" @click="clearData">Upload More Files</button>
      </form>
      <div v-if="processingFile" class="lds-dual-ring"></div>
      <p v-bind:class="{ invisible: supportedFileFormat }" class="invalid-file">Please provide us with <b>.PDF</b> file format</p>
    </div>
    <div class="files-container">
      <div v-for="(file, key) in files" v-bind:key="key" class="file-container"> 
      <!-- <img class="preview" v-bind:ref="'preview'+parseInt( key )"/> -->
        <b>{{ file.name.length>25 ? `${file.name.substring(0, 23)}...`: file.name}}</b>
        <div class="remove-container">
          <a class="remove" v-on:click="removeFile( key )">Delete</a>
        </div>
      </div>
      <a class="submit-button" v-on:click="submitFiles()" v-show="files.length > 0">Submit</a>
    </div>
  </div>
</template>

<script>

import axios from 'axios';

//const liveUrl = 'https://87.99.88.126:5550/upload';
const liveUrl = 'https://files.adverts.lv:5550/upload';

export default {
  name: 'FileUpload',
  props: {
    msg: String
  },
  data(){
    return {
      dragAndDropCapable: false,
      files: [],
      response: '',
      email: null,
      fileSent: false,
      dropIconInvisible: false,
      supportedFileFormat: true,
      emailProvided: false,
      processingFile: false
    }
  },
  methods: {
     upload(event){
      let data = new FormData();
      let file = event.target.files[0];

      data.append('name', 'my-file')
      data.append('file', file)

      
    },
    async submitFiles(err){
      if(this.email) {
        this.processingFile = false;
        this.emailProvided = false;
        /*
          Initialize the form data
        */
        let formData = new FormData();
        /*
          Iteate over any file sent over appending the files
          to the form data.
        */
        for( var i = 0; i < this.files.length; i++ ){
          let file = this.files[i];
          formData.append('files[' + i + ']', file);
        }
        formData.append('email', this.email);
        try {
          const res = await axios.post(liveUrl,
          formData,
          {
            headers: {
                'Content-Type': 'multipart/form-data'
            }
          });
          this.response = res['data'];
          if(res['data'] == 'File uploaded!') {
            this.fileSent = true;
            this.files = [];
            this.processingFile = false;
          }
        } catch {
          this.processingFile = false;
          this.response = err;
        }
      } else {
        this.emailProvided = true;
      }
    },
    determineDragAndDropCapable(){
    const div = document.createElement('div');
      /*
        Check to see if the `draggable` event is in the element
        or the `ondragstart` and `ondrop` events are in the element. If
        they are, then we have what we need for dragging and dropping files.

        We also check to see if the window has `FormData` and `FileReader` objects
        present so we can do our AJAX uploading
      */
    return ( ( 'draggable' in div )
            || ( 'ondragstart' in div && 'ondrop' in div ) )
            && 'FormData' in window
            && 'FileReader' in window;
    },
    async removeFile(key) {
      this.files.splice( key, 1 );
      
      if(this.files.length < 1){
        this.dropIconInvisible = false;
        await this.$nextTick;
      }
    },
    handleFileUpload(){
      for( let i = 0; i < this.$refs.file.files.length; i++ ){
        this.files.push( this.$refs.file.files[i] )
        this.checkFile();
      }
    },
    addFile(e){
      for( let i = 0; i < e.dataTransfer.files.length; i++ ){
        //Check if fail is supported format(XML, PDF)
          this.files.push( e.dataTransfer.files[i] );
          this.checkFile();
      }
    },
    clearData(){
      setTimeout(() => {
        this.files = [];
        this.fileSent = false;
      }, 20);
      
      
    },
    checkFile(){
      this.supportedFileFormat= true;
      this.files.forEach(file => {
        if (/\.(pdf|xml)/.test(file.name)) {
          this.dropIconInvisible = true;
        } else {
          this.supportedFileFormat= false;
        }
      });
      if(this.supportedFileFormat === false) {
        this.clearData();
      }
      //this.supportedFileFormat ? true : this.files.length = 0;
    }
  },
  mounted(){
    this.supportedFileFormat= true;
    this.dragAndDropCapable = this.determineDragAndDropCapable();

    //If drag and drop capable, then we continue to bind events to our elements.
    if( this.dragAndDropCapable ){

      ['drag', 'dragstart', 'dragend', 'dragover', 'dragenter', 'dragleave', 'drop'].forEach( function( evt ) {
        /*
          For each event add an event listener that prevents the default action
          (opening the file in the browser) and stop the propagation of the event (so
          no other elements open the file in the browser)
        */
        this.$refs.fileform.addEventListener(evt, function(e){
          e.preventDefault();
          e.stopPropagation();
        }.bind(this), false);
      }.bind(this));

      this.$refs.fileform.addEventListener('drop', function(e){
        
        for( let i = 0; i < e.dataTransfer.files.length; i++ ){
          this.files.push( e.dataTransfer.files[i] );
          this.checkFile();
        }
      }.bind(this));
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>

  .drop-form {
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
    align-items: center;
    width: 28em;
    height: 20em;
    background: rgb(214, 255, 214);

    &::after {
      margin: 1em 1em;
      content: '';
      position: absolute;
      left: 0px;
      height: 14em;
      width: 25.8em;
      border: dashed $border-color 2px;
    }
  }
  #file {
    display: none;
  }
  .file-select {
    position: relative;
    top: -10px;
    z-index: 10;
  }
  .file-select > .select-button {
    padding: 0.4em;
    z-index: 100;
    color: white;
    background-color: $button-color;
    text-align: center;
    font-weight: bold;
    cursor: pointer;
    &:hover {
      background: rgb(27, 170, 46);
    }
  }
  .reset-btn {
    z-index: 10;
    padding: 0.4em;
    z-index: 100;
    color: white;
    background-color: $button-color;
    text-align: center;
    font-weight: bold;
  }
  .drop-email {
    position: relative;
    top: -1.5em;
    width: 20em;
    height: 2em;
    text-align: center;
    font-size: 0.9em;
    border: solid $border-color 2px;
  }
  .missingEmail::placeholder{
    color:red;
    opacity: 1;
  }
  .drop-icon {
    position: relative;
    width: 4.3em;
    height: 4.3em;
  }
  .files-container {
    position: absolute;
    top: 56%;
    left: 50%;
    transform: translate(-50%, -90%);
    z-index: 30;
   
    overflow: hidden;
  }
  .file-container {
    display: flex;
    justify-content: space-between;
    align-items: flex-start;
    height: 1.4em;
    border-bottom: 1px solid gray;
  }
  div.files-listing {
    img {
      height: 100px;
    }
  }
  div.remove-container{
    text-align: center;
    margin-left: 2em;
  }

  div.remove-container {
    a {
      background: red;
      color: white;
      cursor: pointer;
      margin: 2px;
      padding: 1px 5px;
    }
  }
  a.submit-button{
    display: block;
    position: relative;
    margin: auto;
    margin-top: 0.3em;
    text-align: center;
    width: 14em;
    padding: 10px;
    text-transform: uppercase;
    background-color: $button-color;
    color: white;
    font-weight: bold;
    cursor: pointer;
    
    &:hover {
      background: rgb(27, 170, 46);
    }
  }
  .active {
    height: 5em;
    background: white;
    border-radius: 50%;
    margin: -1.0em;
    fill: rgb(62, 64, 88);
  }
  .green {
    b {
      color: rgb(62, 64, 88);
    }
  }
  .invisible {
    visibility: hidden;
  }
  .invalid-file {
    color: red;
    font-size: 1.2em;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, 40%);
  }
  // Loading icon

  .lds-dual-ring {
    position: absolute;
    z-index: 90;
    left: 50%;
    top: 30%;
    transform: translate(-50%, 50%);
    display: inline-block;
    width: 64px;
    height: 64px;
  }
  .lds-dual-ring:after {
    content: " ";
    display: block;
    width: 46px;
    height: 46px;
    margin: 1px;
    border-radius: 50%;
    border: 5px solid rgb(17, 85, 43);
    border-color: rgb(28, 100, 52) transparent rgb(28, 100, 52) transparent;
    animation: lds-dual-ring 1.2s linear infinite;
  }
  @keyframes lds-dual-ring {
    0% {
      transform: rotate(0deg);
    }
    100% {
      transform: rotate(360deg);
    }
  }
  .cover {
    position: absolute;
    width: 40em;
    height: 20em;
    background: rgba(246, 255, 246,0.46);
    z-index: 50;
  }
</style>
