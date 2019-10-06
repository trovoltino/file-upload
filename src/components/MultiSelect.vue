<template>
  <div class="multi-select">
    <div class="selected-option" @click="isActive = true">{{selectedOption}}</div>
    <ul v-if="isActive" class="options-container">
      <li class="option" @click="selectManager(manager)" v-for="(manager, index) in managerList" :key="index">{{manager}}</li>
      <li class="option" style="margin-top: 1.6em;" @click="selectManager('File-Check Only')">File-Check Only</li>
    </ul>
  </div>
</template>

<script>
import axios from 'axios';
//const liveUrl = 'http://localhost:5555/';
const liveUrl = 'https://files.adverts.lv:5550/';
export default {
  name: 'MultiSelect',
  data(){
    return {
      isActive:false,
      selectedOption: 'Chose sales-manager or File-Check only',
      managerList: ''
    }
  },
  methods: {
    selectManager(manager) {
      this.selectedOption = manager;
      this.isActive = false;
      this.$emit('managerSelected', manager);
    }
  },
  async created() {
    const getManagers = await axios.get(`${liveUrl}managers`, {
            responseType: 'json'
        });
      this.managerList = getManagers.data;
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped>
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    list-style: none;
  }
  .multi-select {
    display: flex;
    flex-direction: column;
    justify-content: space-evenly;
    align-items: center;
    .options-container{
      z-index: 110;
      position: absolute;
      top: 2em;
      left: 50%;
      transform: translate(-50%);
      width: 18.6em;
    }
    .option {
      padding: 5px 3em;
      border-bottom: 1px rgb(200, 200, 200) solid;
      background: white;
      text-align: right;
      &:hover {
        background: $border-color;
      }
    }
    .selected-option {
      background: white;
      width: 100%;
      font-size: 0.9em;
      border: solid $border-color 2px;
      height: 1.4em;
      padding: 0.2em 0.3em;
      display: flex;
      align-items: center;
      justify-content: center;
      &:hover {
        border: solid $button-active 2px;
      }
      &::after {
        content: '';
        height: 1em;
        width: 1em;
        background: $button-color;
        position: absolute;
        right: 21.4%;
        top: 6.6%;
        clip-path: polygon(50% 100%, 9% 0, 91% 0);
      }
    }
  }
</style>
