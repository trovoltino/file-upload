<template>
  <div class="file-upload">
    <div id="file-drag-drop">
      <form ref="fileform" class="drop-form">
        <input v-model="email" type="text" placeholder="Please enter your Email" class="drop-email">
        <img src="../assets/images/download.svg" class="drop-icon" alt="">
        <span class="drop-files" name="sampleFile"><b>Choose a file</b> and drag it here.</span>
        <input type="file"  @change="upload($event)" id="file-input">
        <span class="post-msg">{{response}}</span>
      </form>
    </div>
    <div v-for="(file, key) in files" v-bind:key="key" class="file-listing"> 
      <img class="preview" v-bind:ref="'preview'+parseInt( key )"/>
      {{ file.name }}
      <div class="remove-container">
        <a class="remove" v-on:click="removeFile( key )">Remove</a>
      </div>
    </div>
    
    <a class="submit-button" v-on:click="submitFiles()" v-show="files.length > 0">Submit</a>
  </div>
</template>

<script>

import axios from 'axios';
const url = 'http://localhost:5555/upload';
//const url = 'http://127.0.0.1:5000/GetEmailandPDF';

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
      email:''
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
        formData.append('email', this.email);
      }

      try {
          const res = await axios.post(url,
          formData,
          {
            headers: {
                'Content-Type': 'multipart/form-data'
            }
          });
          this.response = res['data'];
        } catch {
          this.response = err;
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
    removeFile( key ){
      this.files.splice( key, 1 );
    },
    getImagePreviews(){
    /*
      Iterate over all of the files and generate an image preview for each one.
    */
      for( let i = 0; i < this.files.length; i++ ){
        /*
          Ensure the file is an image file
        */
        if ( /\.(jpe?g|png|gif)$/i.test( this.files[i].name ) ) {

          let reader = new FileReader();
          /*
            Add an event listener for when the file has been loaded
            to update the src on the file preview.
          */
          reader.addEventListener("load", function(){
            this.$refs['preview'+parseInt( i )][0].src = reader.result;
          }.bind(this), false);
          /*
            Read the data for the file in through the reader. When it has
            been loaded, we listen to the event propagated and set the image
            src to what was loaded from the reader.
          */
          reader.readAsDataURL( this.files[i] );
        }else{
          /*
            We do the next tick so the reference is bound and we can access it.
          */
          this.$nextTick(function(){
            this.$refs['preview'+parseInt( i )][0].src = '../assets/images/fewclicks.png';
          });
        }
      }
    }
  },
  mounted(){
  
    this.dragAndDropCapable = this.determineDragAndDropCapable();

    //If drag and drop capable, then we continue to bind events to our elements.
    if( this.dragAndDropCapable ){
      /*
        Listen to all of the drag events and bind an event listener to each
        for the fileform.
      */
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

      //Add an event listener for drop to the form
      this.$refs.fileform.addEventListener('drop', function(e){
        /*
          Capture the files from the drop event and add them to our local files
          array.
        */
        for( let i = 0; i < e.dataTransfer.files.length; i++ ){
          this.files.push( e.dataTransfer.files[i] );
        }
        this.getImagePreviews();
      }.bind(this));
    }
  },
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
  .drop-form {
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
    align-items: center;
    width: 40em;
    height: 20em;
    background: rgb(206, 216, 246);
  }
  .drop-email {
    position: relative;
    top: -2em;
    width: 20em;
    height: 2em;
    text-align: center;
    font-size: 0.9em;
  }
  .drop-form::after {
    margin: 1em 1em;
    content: '';
    position: absolute;
    left: 0px;
    height: 14em;
    width: 38em;
    border: dashed rgb(79, 137, 195) 2px;
  }
  .drop-icon {
    content: '';
    position: relative;
    width: 4.3em;
    height: 4.3em;
  }
  div.file-listing img{
    height: 100px;
  }
  div.remove-container{
  text-align: center;
  }

  div.remove-container a{
    background: red;
    color: white;
    cursor: pointer;
    margin: 2px;
    padding: 1px 5px;
  }
  a.submit-button{
    display: block;
    margin: auto;
    text-align: center;
    width: 200px;
    padding: 10px;
    text-transform: uppercase;
    background-color: rgb(138, 161, 236);
    color: white;
    font-weight: bold;
    margin-top: 20px;
    cursor: pointer;
  }
</style>
