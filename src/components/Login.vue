<template>
  <v-app id="inspire">
    <v-content>
      <v-container fluid fill-height>
        <v-layout align-center justify-center>
          <v-flex xs12 sm8 md4>

            <v-card class="elevation-12">

              <v-toolbar dark color="primary">
                <v-toolbar-title>Авторизация</v-toolbar-title>
                <v-spacer></v-spacer>
              </v-toolbar>

              <v-card-text>
                <v-form>
                  <v-text-field
                    v-model="formData.login"
                    prepend-icon="person" name="login" label="Login" type="text"
                  ></v-text-field>
                  <v-text-field
                    v-model="formData.password"
                    id="password" prepend-icon="lock"
                    name="password" label="Password" type="password"
                  ></v-text-field>
                </v-form>

                <p v-if="errorTypedAccessData" class="errorAccessData">
                  Неверный Логин или Пароль</p>
              </v-card-text>

              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn @click="tryLogin()" color="primary">Войти</v-btn>
              </v-card-actions>
            </v-card>

          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
export default {
  data: () => ({
    drawer: null,
    formData: { login: '', password: '' },
    accessData: { login: 'masha', password: 'pass' },
    errorTypedAccessData: false
  }),
  props: {
    // source: String
  },
  methods: {
    tryLogin () {
      if (this.accessData.login === this.formData.login &&
        this.accessData.password === this.formData.password) {
        sessionStorage.setItem('agroAdminAuth', 'isAuth')
        this.$emit('userIsLogIn')
      } else {
        this.errorTypedAccessData = true
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  .errorAccessData {
    color: red;
  }
</style>
