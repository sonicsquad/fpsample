<template>
  <div   class="section container">
  <span>Please note: this is a demo. Nothing is stored server side.</span>
  <br/>
  <br/> 
  <template v-if="!isAuthenticated">
    
      <input v-model="username" placeholder="Username" @input="checkIsRegistered()" autocomplete="webauthn username" />
    
    
      <input type="checkbox" v-model="isRoaming">
    
    <div>
      <button type="is-primary" @click="register()" :disabled="!username">Register device</button>
      <button type="is-primary" @click="login()" :disabled="!isRegistered">Authenticate</button>
    </div>
  </template>
  <template v-if="isAuthenticated">
    <p>{{'Hello ' + username + '!'}}</p>
    <section v-if="registrationData && !authenticationData" style="text-align: left;">
      <p><b>Credential ID:</b> {{registrationData.credential.id}}</p>
      <p><b>Public Key:</b> {{registrationData.credential.publicKey.substring(0,32)}}...</p>
      <p><b>Algorithm:</b> {{registrationData.credential.algorithm}}</p>
      <p><b>Authenticator Name:</b> {{registrationData.authenticator.name}}</p>
    </section>
    <section v-if="authenticationData" style="text-align: left;">
      <p><b>Credential ID:</b> {{authenticationData.credentialId}}</p>
      <p><b>Signature:</b> {{authenticationData.signature.substring(0,32)}}...</p>
    </section>
    <div>
      <b-button type="is-primary" @click="logout()">Sign Out</b-button>
    </div>
  </template>
</div>
</template>

<script>
import { client, parsers } from '@passwordless-id/webauthn'

export default {
  data() {
    return {
    username: null,
    isRegistered: false,
    isAuthenticated: false,
    isRoaming: false,
    registrationData: null,
    authenticationData: null
    }
  },
  methods: {
    async checkIsRegistered() {
      console.log(this.username + ' => ' + !!window.localStorage.getItem(this.username))
      this.isRegistered = !!window.localStorage.getItem(this.username)
    },
    async register() {
      let res = await client.register(this.username, window.crypto.randomUUID(), { authType: this.isRoaming ? 'roaming' : 'auto' })
      console.debug(res)

      const parsed = parsers.parseRegistration(res)
      console.log(parsed)

      window.localStorage.setItem(this.username, parsed.credential.id)
      this.isAuthenticated = true
      this.registrationData = parsed

      this.$buefy.toast.open({
        message: 'Registered!',
        type: 'is-success'
      })

      await this.checkIsRegistered()
    },
    async login() {
      let credentialId = window.localStorage.getItem(this.username)
      let res = await client.authenticate(credentialId ? [credentialId] : [], window.crypto.randomUUID(), { authType: this.isRoaming ? 'roaming' : 'auto' })
      console.debug(res)

      const parsed = parsers.parseAuthentication(res)
      console.log(parsed)

      this.isAuthenticated = true
      this.authenticationData = parsed

      
    },
    async logout() {
      this.isAuthenticated = false;
      
      this.authenticationData = null
      this.registrationData = null
    }
  },
  
  mounted(){
    
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
 