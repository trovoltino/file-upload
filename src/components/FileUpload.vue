<template>
  <div class="file-upload">
    <MultiSelect v-on:managerSelected="onSelectChange" class="manager-select"/>
    <div v-if="processingFile" class="cover"></div>
    <div id="file-drag-drop">
      <form ref="fileform" class="drop-form">
        <input v-model="email" v-bind:class="{ missingEmail: emailProvided }" v-bind:style="isManagerSelected ? 'bottom: 0.4em;' : 'bottom: -0.4em;'" type="email" placeholder="Please enter your Email to receve results" class="drop-email">
        <textarea class="comments" v-if="isManagerSelected" v-model="comments" cols="30" rows="10" placeholder="Comments (if necessary)"></textarea>
        <img v-if="!fileSent" v-bind:class="{ invisible: dropIconInvisible }" src="@/assets/images/download.svg" class="drop-icon" alt="drag and drop">
        <img v-if="fileSent" v-bind:class="{ active: fileSent }" src="@/assets/images/checked.svg" alt="">
        <span v-if="!fileSent" v-bind:class="{ invisible: !supportedFileFormat }" name="sampleFile"><b>Choose PDF files</b> and drag it here.</span>
        <span v-else class="green">
          <b>File has been uploaded successfully!</b>
          <br><p v-if="isManagerSelected">And will be processed shortly.</p>
              <p v-else >We will send results to {{this.email}} shortly.</p>
        </span>
        <label v-if="!fileSent" class="file-select">
          <div class="select-button">Select File</div>
          <input  type="file" id="file" ref="file" multiple v-on:change="handleFileUpload()"/>
        </label>
        <button v-else class="reset-btn" @click="clearData">Upload More Files</button>
      </form>
      <div v-if="processingFile" class="lds-dual-ring"></div>
      <p v-bind:class="{ invisible: supportedFileFormat }" class="invalid-file">Please provide us with <b>.PDF</b> file format</p>
    </div>
    <div v-bind:style="files.length>2 ? 'width: 22em;' : 'width: 20em;' " class="files-container">
      <div class="files-limiter">
        <div v-for="(file, key) in files" v-bind:key="key" class="file-container"> 
      <!-- <img class="preview" v-bind:ref="'preview'+parseInt( key )"/> -->
        <b>{{ file.name.length>28 ? `${file.name.substring(0, 26)}...`: file.name}}</b>
        <div class="remove-container">
          <a class="remove" v-on:click="removeFile( key )">Delete</a>
        </div>
      </div>
      </div>
      <a class="submit-button" v-on:click="submitFiles()" v-show="files.length > 0">Submit</a>
    </div>
  </div>
</template>

<script>

import axios from 'axios';
import MultiSelect from '@/components/MultiSelect';
//const liveUrl = 'http://localhost:5555/';
//const demoUrl = 'https://files-uploads.herokuapp.com/upload';
const liveUrl = 'https://files.adverts.lv:5550/';

export default {
  name: 'FileUpload',
  components: {
    MultiSelect
  },
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
      processingFile: false,
      preSelect: null,
      comments: '',
      isManagerSelected: false,
      managerSelected :''
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
        this.processingFile = true;
        this.emailProvided = false;
        /*
          Initialize the form data
        */
        let formData = new FormData();
        /*
          Iteate over any file sent over appending the files
          to the form data.
        */
        if (this.isManagerSelected) {
          formData.append('email', this.managerSelected);
          formData.append('clientEmail',this.email);
          formData.append('comments', this.comments);
        } else {
          formData.append('email', this.email);
        }
        for( var i = 0; i < this.files.length; i++ ){
          let file = this.files[i];
          formData.append('files[' + i + ']', file);
        }
        
        try {
          axios.post(`${liveUrl}upload`,
          formData,
          {
            headers: {
                'Content-Type': 'multipart/form-data'
            }
          }).then((res) => {
            this.response = res['data'];
            if(res.status == 200) {
              this.fileSent = true;
              this.files = [];
              this.processingFile = false;
              this.comments = '';
            }
          });
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
    onSelectChange(manager) {
      if (manager=='File-Check Only') {
        this.isManagerSelected = false;
      } else {
        this.managerSelected = manager;
        this.isManagerSelected = true;
      }
      
    },
    clearData(){
      this.files = [];
      this.fileSent = false;
      this.dropIconInvisible = false;
      this.comments = '';
    },
    checkFile(){
      this.supportedFileFormat= true;
      this.files.forEach(file => {
        if (/\.(pdf|xml|PDF|Pdf)/.test(file.name)) {
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
  .file-upload {
    background: rgb(214, 255, 214);
    padding-top: 1em;
  }
  .drop-form {
    position:relative;
    bottom: 0px;
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    align-items: center;
    width: 32em;
    height: 20em;

    &::after {
      margin: 1em 1em;
      content: '';
      position: absolute;
      bottom: 0px;
      left: 0px;
      height: 14em;
      width: 30em;
      border: dashed $border-color 2px;
    }
  }
  #file {
    display: none;
  }
  .file-select {
    position: relative;
    bottom: 1.5em;
    z-index: 10;
  }
  .file-select > .select-button {
    padding: 0.4em;
    z-index: 100;
    color: white;
    background-color: $button-color;
    text-align: center;
    font-weight: bold;
    position: relative;
    bottom: 0.8em;
    cursor: pointer;
    &:hover {
      background: $button-active;
    }
  }
  .reset-btn {
    position: relative;
    top: -2em;
    padding: 0.4em;
    z-index: 100;
    color: white;
    background-color: $button-color;
    text-align: center;
    font-weight: bold;
  }
  .drop-email {
    position: relative;
    bottom: -0.4em;
    width: 20.4em;
    height: 2em;
    text-align: center;
    font-size: 0.9em;
    border: solid $border-color 2px;
  }
  .comments{
    position: absolute;
    top: 2.4em;
    width: 20.2em;
    height: 2em;
    font-size: 1.1em;
    border: solid $border-color 2px;
    z-index: 40;
  }
  .comments::placeholder {
    text-align: center;
  }
  .comments:hover {
    height: 6em;
  }
  .manager-select { 
    z-index: 10;
    text-align: center;
    margin: auto;
    margin-bottom: 1em;
    width: 18em;
    background: white;

  }
  .missingEmail::placeholder{
    color:red;
    opacity: 1;
  }
  .drop-icon {
    position: relative;
    bottom: -1em;
    width: 4.3em;
    height: 4.3em;
  }
  .files-container {
    position: absolute;
    bottom: 34%;
    right: 50%;
    -webkit-transform: translate(50%, 10%);
         -moz-transform: translate(50%, 10%);
         -ms-transform: translate(50%, 10%);
         -o-transform: translate(50%, 10%);
         transform: translate(50%, 10%);
    
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
  .files-limiter {
    overflow: auto;
    max-height: 4.2em;
  }
  div.files-listing {
    img {
      height: 100px;
    }
  }
  div.remove-container{
    text-align: center;
    margin-left: 1em;
  }

  div.remove-container {
    a {
      background: $button-color;
      color: white;
      cursor: pointer;
      margin: 2px;
      padding: 1px 5px;
      &:hover {
        background: $button-active;
      } 
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
      background: $button-active;
    }
  }
  .active {
    height: 5em;
    background: white;
    border-radius: 50%;
    margin: 2.4em;
    fill: rgb(62, 64, 88);
  }
  .green {
    b {
      color: rgb(62, 64, 88);
    }
    width: 70%;
    position: relative;
    bottom: 1.7em;
  }
  .invisible {
    visibility: hidden;
  }
  .invalid-file {
    color: red;
    font-size: 1.2em;
    position: absolute;
    left: 50%;
    top: 55%;
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
    top: -1em;
    width: 32em;
    height: 100%;
    background: rgba(246, 255, 246,0.46);
    z-index: 50;
  }
</style>
