<template>
  <q-page class="flex flex-center">
    <q-card
      square
      style="width: 400px; padding:50px"
    >
      <q-card-section>
        <div class="text-h6">
          {{ lang.auth.login.login }}
        </div>
      </q-card-section>

      <q-form
        class="q-gutter-md"
        @submit="onSubmit"
      >
        <q-card-section>
          <q-input
            id="email"
            v-model.trim="data.body.email"
            type="email"
            :label="lang.auth.email"
            :rules="validations['email']"
            lazy-rules
            autofocus
          />
          <q-input
            id="password"
            v-model="data.body.password"
            type="password"
            :label="lang.auth.login.password"
            :rules="validations['password']"
            lazy-rules
          /><br>
          <q-checkbox
            id="rememberMe"
            v-model="data.rememberMe"
            :label="lang.auth.login.rememberMe"
          />
        </q-card-section>
        <q-card-actions>
          <q-btn
            :label="lang.auth.login.login"
            color="primary"
            :loading="loading"
            type="submit"
          />
        </q-card-actions>
      </q-form>
      <router-link to="/password/forgot">
        <a>{{ lang.auth.login.passwordForgot }}</a>
      </router-link>

      <q-card-section>
        <a>{{ lang.auth.login.registerMessage }} </a>
        <router-link to="/register">
          <a>{{ lang.auth.login.register }}</a>
        </router-link>
      </q-card-section>
    </q-card>
  </q-page>
</template>

<script>
import isEmail from 'validator/lib/isEmail'

export default {
  name: 'Login',
  data () {
    return {
      lang: {
        auth: {}
      },
      data: {
        body: {
          email: '',
          password: ''
        },
        rememberMe: false
      },
      loading: false,
      validations: {
        email: [
          val => !!val || this.lang.auth.validations.required,
          val => isEmail(val) || this.lang.auth.validations.email
        ],
        password: [val => !!val || this.lang.auth.validations.required]
      }
    }
  },
  watch: {
    '$q.lang.isoName' (val) {
      this.__setupLang()
    }
  },
  beforeMount () {
    this.__setupLang()
  },
  methods: {
    __setupLang () {
      let isoName = this.$q.lang.isoName || 'en-us'
      let lang
      try {
        lang = require(`auth-token-based/lang/${isoName}`)
      } catch (e) { }

      if (lang !== void 0) {
        this.lang['auth'] = { ...lang.default.auth }
      }
    },
    onSubmit () {
      this.loading = true
      this.$auth
        .login(this.data)
        .then(response => {
          this.$router.push('/')
          this.$store.dispatch('auth/loginCallback')
        })
        .catch(error => {
          if (error.response) {
            if (error.response.status === 401) {
              this.$q.dialog({
                message: this.lang.auth.login.verificationRequired
              })
            } else if (error.response.status === 403) {
              this.$q.dialog({
                message: this.lang.auth.login.invalidCredentials
              })
            } else {
              console.error(error)
            }
          }
        })
        .finally(() => {
          this.loading = false
        })
    }
  }
}
</script>
