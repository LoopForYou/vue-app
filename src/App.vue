<template>
  <div id="app">
  <!-- <textarea class="ui-hidden-accessible" id="myTextArea" v-model="scannedData" ref="scannedDataInput" @blur="focusInput"/> -->
  <div v-if="!loading">
  	<ID v-if="scanType === 'pdf417'" :data="data" :age="age" :expired="expired" :expiredFor="expiredFor" :expiresIn="expiresIn" />
  	<UPC v-if="scanType === 'UPC'" :upc="UPCdata" />
  </div>
    <Loading v-if="loading"/>
  </div>
</template>

<script>
import Vue from 'vue'
import * as moment from "moment/moment";
import Loading from '@/components/Loading.vue'
import ID from '@/components/ID.vue'
import UPC from '@/components/UPC.vue'
//import VueBarcodeScanner from '@/utils/scanner.js'
//Vue.use(VueBarcodeScanner)
export default {
 components: {
    Loading,
    ID,
    UPC
  },
    data: () => ({
      loading: true,
      scannedData: '',
      pdf417Arr: '',
      data:{},
      UPCdata:'',
      age: '',
      expired:'',
      expiredFor:'',
      expiresIn:'',
      scanType: '',
      timer: null
    }),
    mounted () {
        document.addEventListener("keyup", event => {this.getInput(event)});
    },
    destroyed () {
      // Remove listener when component is destroyed
      document.removeEventListener("keyup", console.log("Listening removed"));
     
    },
    methods: {
    checkAge(birthdate){
    this.age = '';
    	this.age = moment().diff(birthdate, 'years');    	
    },
    checkExpiration(expiration){
    this.expiresIn = '';
    this.expiredFor = '';
    this.expired = '';
    	if (moment().diff(expiration, 'days') < 0){
    		this.expiresIn = moment().diff(expiration, 'days') * -1;
    		this.expired = false;
    	} else {
    		this.expiredFor = moment().diff(expiration, 'days');
    		this.expired = true;
    	
    	}
    	    	
    },
      // Create callback function to receive barcode when the scanner is already done
    delay(fn, ms) {
      	let timer = 0
      	return function(...args) {
      		clearTimeout(timer)
      		timer = setTimeout(fn.bind(this, ...args), ms || 0)
    	}
	},
	addTextAreaCallback(textArea, callback, delay) {
    	var timer = null;
    	textArea.onkeypress = function() {
        	if (timer) {
            	window.clearTimeout(timer);
        	}
        	timer = window.setTimeout( function() {
            	timer = null;
            	callback(textArea);
        	}, delay );
    	};
    	textArea = null;
	},
	scan() {
	this.startParse = true;	
	this.scanType = '';
	this.pdf417Arr = '';
	this.UPCdata = '';
	if(this.scannedData.includes("ANSI")){
		this.pdf417Arr = this.scannedData.split('\n');
		for (var i = 0; i < this.pdf417Arr.length; i++) {
			if (this.pdf417Arr[i].slice(0, 3) == 'DBB' || this.pdf417Arr[i].slice(0, 3) == 'DBA'){
				this.data[this.pdf417Arr[i].slice(0, 3)] = moment(this.pdf417Arr[i].slice(3),'MMDDYYYY');
				 if (this.pdf417Arr[i].slice(0, 3) == 'DBB'){
				 	this.checkAge(moment(this.pdf417Arr[i].slice(3),'MMDDYYYY'));
				 } else {
				 	this.checkExpiration(moment(this.pdf417Arr[i].slice(3),'MMDDYYYY'));
				 }
			} else {
				this.data[this.pdf417Arr[i].slice(0, 3)] = this.pdf417Arr[i].slice(3);
			}
		}
		this.scanType = 'pdf417';
		this.loading = false;
		this.pdf417 = '';
	}
	
	if (!this.scannedData.includes("ANSI")) {
		this.UPCdata = this.scannedData;
		this.scanType = 'UPC';
		this.loading = false;
	}
	this.loading = false;
	this.scannedData = '';
	},
	focusInput() {
		this.$refs.scannedDataInput.focus();
    },
     clearObject() {
    	return {};
    },
    getInput (e) {
    	var timer = null;
    	this.loading = true;
        this.scannedData += String.fromCharCode(e.which);
        clearTimeout(this.timer);
        this.timer = setTimeout( () => this.scan(), 1000 )
       
}
	}
  }
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
textarea{
   -webkit-user-select: auto;
} 
/*.ui-hidden-accessible {
  position: absolute !important;
  height: 1px;
  width: 1px;
  overflow: hidden;
  clip: rect(1px,1px,1px,1px);
} */
</style>
