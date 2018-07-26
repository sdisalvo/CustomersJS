<template>
  <v-app>
      <v-toolbar app :clipped-left="clipped">
          <v-toolbar-title v-text="title"></v-toolbar-title>
          <v-spacer></v-spacer>
          <v-avatar><img v-bind:src="profileimg" v-if="seen"></v-avatar>
          <v-btn flat small v-on:click="login">{{ logintxt }}</v-btn>
      </v-toolbar>
    <v-content>
        <div id="divcontent" v-if="seen" >
            <Data />
        </div>
    </v-content>
    <v-footer :fixed="fixed" app>
      <span>&copy; 2018 Mediolanum</span>
    </v-footer>
  </v-app>
</template>

<script>
  import Data from './components/Data'

  export default {
    name: 'App',
    components: {
        Data
    },
    data () {
        return {
            clipped: false,
              drawer: true,
              fixed: false,
              items: [{
                icon: 'bubble_chart',
                title: 'Inspire'
              }],
              miniVariant: false,
              right: true,
              rightDrawer: false,
            title: 'AWS Serverless Sample',
            logintxt: "Login",
            seen: false,
            profileimg: null
         }
        },
        mounted: function () {
            this.init();
        },
        methods: {
            init: function () {
                gapi.load('client:auth2', this.initClient);
            },
            initClient: function () {
                var self = this;

                // Initialize the gapi.client object, which app uses to make API requests.
                // Get API key and client ID from API Console.
                // 'scope' field specifies space-delimited list of access scopes.
                try {
                    gapi.client.init({
                        'apiKey': 'AIzaSyB966ZrrLroBnR6LIz9WbOONWRWWoIKTzI',
                        'clientId': '105248247635-ag3j6qvnknpsmo8a44kr0ffvfkr5q4ph.apps.googleusercontent.com',
                        'scope': "email profile"
                    }).then(function () {
                        var GoogleAuth = gapi.auth2.getAuthInstance();
                        // Listen for sign-in state changes.
                        GoogleAuth.isSignedIn.listen(self.updateSigninStatus);
                        self.updateSigninStatus();
                    });
                } catch (err) {
                    this.logintxt = "Login error!";
                    console.log( err );
                }
                this.updateSigninStatus();
            },
            updateSigninStatus: function () {
                var GoogleAuth = gapi.auth2.getAuthInstance();
                var user = GoogleAuth.currentUser.get();
                var isAuthorized = user.hasGrantedScopes("email profile");
                if (isAuthorized) {
                    this.logintxt = "Logout";
                    this.seen = true;
                    var profile = user.getBasicProfile();
                    this.profileimg = profile.getImageUrl();
                } else {
                    this.logintxt = "Login";
                    this.seen = false;
                }
            },
            login: function () {
                var GoogleAuth = gapi.auth2.getAuthInstance();
                if (GoogleAuth.isSignedIn.get()) {
                    // User is authorized and has clicked 'Sign out' button.
                    GoogleAuth.signOut();
                } else {
                    // User is not signed in. Start Google auth flow.
                    GoogleAuth.signIn();
                }
            }
        }
  }
</script>
