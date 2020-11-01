<template>


        <StackLayout dock="top" height="90%" width="100%" style="padding:30;">

            <TextField v-model="noteTitle" hint="Write a title" />
            <TextField v-model="noteBody" hint="Write a note" style="height:100;"/>
            <Button text="Submit Post" @tap="submitPost()"></Button>
            
            <Label v-if="needLocation" text="Looking up your location..." />
            <Label v-if="locationFailure" text="Can't find current location" />
            <Label v-if="location" :text="locationDescription" textWrap="true" />
        </StackLayout>

</template>
<script>
    import * as Geolocation from 'nativescript-geolocation';
    const httpModule = require("http");
    const appSettings = require("tns-core-modules/application-settings");

    import { inappmessaging } from "nativescript-plugin-firebase/inappmessaging";

    export default {
        watch: {},
        created() {
            //check if gps is enabled on the phone
            Geolocation.enableLocationRequest(true)
            .then(() => {
                Geolocation.isEnabled().then(isLocationEnabled => {
                    console.log('result is '+isLocationEnabled);
                    if(!isLocationEnabled) {
                        this.needLocation = false;
                        this.locationFailure = true;
                        // potentially do more then just end here...
                        return;
                    }
                    // MUST pass empty object!!
                    Geolocation.getCurrentLocation({})
                    .then(result => {
                        console.log('loc result', result);
                        this.needLocation = false;
                        this.location = result;
                    })
                    .catch(e => {
                        console.log('loc error', e);
                    });
                });
            });
        },
        data() {
            return {
                needLocation:true,
                locationFailure:false,
                location:null,
                noteTitle:"",
                noteBody:"",
                searchValue: '',
                APIURL: ""
            };
        },
        computed: {
            locationDescription() {
                //display current location
                return `You are at ${this.location.latitude}, ${this.location.longitude}. Your altitude is ${this.location.altitude}.`;
            },
            name(){
                //get the name of the current login user to display in the side drawer
                return appSettings.getString('name','');
            }
        },
        methods: {
            onStart(){
            },
            submitPost(){
                //get all stored variables from the login session
                var userToken = appSettings.getString('userToken',0);
                var appURL = appSettings.getString('appURL',0);
                this.APIURL = appURL;
                httpModule.request({
                    url: appURL+'/api/notes',//"http://192.168.0.83:8000/api/home",
                    method: "Post",
                    headers: { 
                        "Content-Type": "application/json",
                        "Accept": "application/json" ,
                        "Authorization": "Bearer "+userToken
                    },
                    content: JSON.stringify({
                        title:this.noteTitle,
                        description: this.noteBody,
                        lat:this.location.latitude,
                        lng:this.location.longitude
                    })
                }).then(response => {
                    // the post was saved successfully
                    console.log(response);
                    if(response.statusCode === 200){
                        this.noteTitle = '';
                        this.noteBody = '';
                        alert('note added to location');
                    }else{
                        alert('Something has gone wrong'+ response.statusCode);
                    }
                }, error => {
                    console.error(error);
                });
            }
        }
    };
</script>

<style scoped>
</style>