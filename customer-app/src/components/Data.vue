<template>
  <v-card>
    <v-form id="form" v-on:submit.prevent="searchUser">
        <v-layout>
            <v-text-field type="text" v-model="customerkey.customer.customerId" label="Customer Id" />
            <v-text-field type="text" v-model="customerkey.customer.cognome" label="Cognome" />
            <v-text-field type="text" v-model="customerkey.customer.nome" label="Name" />
            <v-btn :loading="loading" color="secondary" type="submit">Search</v-btn>
        </v-layout>
    </v-form>
    <v-data-table
      :headers="headers"
      :items="customers"
      class="elevation-1">
      <template slot="headerCell" slot-scope="props">
        <v-tooltip bottom="">
          <span slot="activator">{{ props.header.text }}</span>
          <span>{{ props.header.text }}</span>
        </v-tooltip>
      </template>
      <template slot="items" slot-scope="props">
        <td>{{ props.item.customerId }}</td>
        <td class="text-xs-center">{{ props.item.cognome }}</td>
        <td class="text-xs-center">{{ props.item.nome }}</td>
        <td class="text-xs-center">{{ props.item.via }}</td>
        <td class="text-xs-center">{{ props.item.numeroCivico }}</td>
      </template>
    </v-data-table>
  </v-card>
</template>

<script>
  export default {
      data () {
          return {
              headers: [
                  { text: 'id', align: 'center', value: 'customerId'  },
                  { text: 'Cognome', align: 'center', value: 'cognome' },
                  { text: 'Nome', align: 'center', value: 'nome' },
                  { text: 'Via', align: 'center', value: 'via' },
                  { text: 'Numero', align: 'center', value: 'numeroCivico' },
              ],
              customers: [],
              customerkey: {
                  customer: {
                      customerId: '',
                      nome: '',
                      cognome: ''
                  }
              },
              loading: false
          }
        },
        methods: {
            searchUser: function () {
                var GoogleAuth = gapi.auth2.getAuthInstance();
                var user = GoogleAuth.currentUser.get();
                var isAuthorized = user.hasGrantedScopes( 'email profile' );

                if (isAuthorized) {
                    this.loading = "true";
                    // Initialize the Amazon Cognito credentials provider
                    AWS.config.region = 'eu-west-1'; // Region
                    AWS.config.credentials = new AWS.CognitoIdentityCredentials({
                        IdentityPoolId: 'eu-west-1:53583a54-5f35-4c13-ad15-9f3f043938ea',
                        Logins: {
                            'accounts.google.com': user.getAuthResponse().id_token
                        }
                    });

                    console.log("Call remote function with data: " + JSON.stringify(this.customerkey));
                    var lambda = new AWS.Lambda({ region: 'eu-west-1' });
                    var pullParams = {
                        FunctionName: 'ci-stack-getCustomerInfo-X6CV8TRD0YAB',
                        InvocationType: 'RequestResponse',
                        LogType: 'None',
                        Payload: JSON.stringify(this.customerkey)
                    };

                    // Call the Lambda function to collect the spin results
                    var self = this;
                    lambda.invoke(pullParams,
                        function (err, data) {
                        if (err) {
                            console.log( "Lambda call error: " + err);
                        } else {
                            console.log("Received data: " + data.Payload);
                            var obj = JSON.parse(data.Payload);
                            self.customers = obj.customers;
                            self.loading = false;
                        }
                    });
                } else {
                    console.log("Not logged !!!");
                }
            }
        }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
</style>
