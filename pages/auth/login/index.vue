<template>
  <v-layout justify-center class="mt-3">
    <v-flex xs12 sm10 md10 lg6>
      <v-form>
        <v-text-field label="EMAIL" v-model="email" outline required></v-text-field>
        <v-text-field
          ref="password"
          v-model="password"
          :type="'password'"
          label="PASSWORD"
          outline
          required
        ></v-text-field>
        <v-btn color="primary" large block @click="submit">Login</v-btn>
        <v-layout row wrap>
          <v-flex xs12 sm4 md4>
            <v-checkbox
              v-model="check"
              label="Remember"
              color="primary"
              value="primary"
              hide-details
            ></v-checkbox>
          </v-flex>
          <v-flex xs12 sm4 md4> </v-flex>
          <v-flex xs12 sm4 md4>
            <v-btn color="primary" to="password" flat class="mt-3">비밀번호 찾기</v-btn>
          </v-flex>
        </v-layout>
      </v-form>
    </v-flex>
  </v-layout>
</template>

<script>
import { mapGetters } from 'vuex'
export default { 
  layout: 'login',
  data () {
    return {
      email: '',
      password: '',
      check: false,
    }
  },
  computed: {
    ...mapGetters('auth', [
      'GET_USER'
    ])
  },
  methods: {
    async submit () {
      try {
        await this.$store.dispatch('auth/LOGIN', {
          email: this.email,
          password: this.password
        })

        this.redirect()
      } catch (err) {
        this.$notifyError("올바르지 않은 이메일주소 혹은 비밀번호입니다.")
      }
    },
     redirect () {
        const {search} = window.location
        const tokens = search.replace(/^\?/, '').split('&')
        let {returnPath} = tokens.reduce((qs, tkn) => {
          const pair = tkn.split('=')
          qs[pair[0]] = decodeURIComponent(pair[1])
          return qs
        }, {})
        if (!returnPath) {
          returnPath = '/'
        }
        this.$router.push(returnPath)
     }
  }
}
</script>
