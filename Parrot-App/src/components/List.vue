<template>

        <RadListView ref="listView"
            for="note in notes"
            pullToRefresh="true"
            :key="index" height="100%"
            backgroundColor="#E8E8E8" separatorColor="transparent"
            id="listView"
            @pullToRefreshInitiated="onPullToRefreshInitiated">

            <v-template>

                <StackLayout paddingTop="5" backgroundColor="#E8E8E8" :id="note.id" @tap="selectNote(note.id)">
                    <StackLayout class="noteContainer">
                        <StackLayout orientation="horizontal"
                            padding="10">
                            <StackLayout>
                                <Label :text="note.title"
                                    class="postAuthotName" />
                                <Label :text="(+note.distance * 100).toFixed(3) +' meters away'"
                                    class="postDateSmall" />
                            </StackLayout>
                        </StackLayout>
                    </StackLayout>
                </StackLayout>

            </v-template>
        </RadListView>


</template>
<script>
    //get external pages and plugins
    const httpModule = require("http");
    const appSettings = require("tns-core-modules/application-settings");
    import * as Geolocation from 'nativescript-geolocation';
    export default {
        computed: {
        },
        watch: {
            location:{
                //if gps location changes reload the notes list, deep watcher is needed
                deep:true,
                handler(){
                    console.log('the gps location has changed, notes list reload')
                    this.getNotes();
                }
            }
        },
        created() {
            //on creation ensure the user has gps enabled otherwise the app will crash
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
            //a delay is is needed to run the inital starting function due to nativescript bug
            setTimeout(() => {
                this.onStart();
            }, 500);
        },
        data() {
            return {
                needLocation:true,
                locationFailure:false,
                location:null,
                mainColor: "#1aa3ff",
                APIURL: "",
                name:"",
                index: 'list',
                notes: [],
            };
        },
        methods: {
            onStart(){
                //this is the first method to be executed on the app starting
                this.name = appSettings.getString('name','');
            },
            getNotes(){
                //make a request to the api server and get all the notes in the area
                console.log('getting notes');
                //get all stored variables from the login session
                var userToken = appSettings.getString('userToken',0);
                var appURL = appSettings.getString('appURL',0);
                this.APIURL = appURL;
                //debuging show 
                console.log(appURL+'/api/notes?lat='+this.location.latitude+'&lng='+this.location.longitude);
                //send request to api server
                httpModule.request({
                    url: appURL+'/api/notes?lat='+this.location.latitude+'&lng='+this.location.longitude,
                    method: "Get",
                    headers: { 
                        "Accept": "application/json" ,
                        "Authorization": "Bearer "+userToken
                    }
                }).then(response => {
                    var result =response.content.toJSON();
                    this.notes = result.notes;
                }, error => {
                    console.error(error);
                });
            }
        }
    };
</script>

<style scoped>
</style>