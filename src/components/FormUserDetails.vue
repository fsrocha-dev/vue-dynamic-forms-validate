<template>
  <div>
    <h1 class="title">Criação de conta</h1>

    <h2 class="subtitle">Crie uma conta ou faça login para solicitar sua assinatura de CoffeeBox</h2>

    <form v-if="!loggedIn" class="form" @input="submit">
      <div class="form-group">
        <label class="form-label" for="email">Email</label>
        <input
          type="text"
          @blur="checkIfUserExists"
          v-model="$v.form.email.$model"
          placeholder="seu@email.com"
          class="form-control"
          id="email"
        />
        <div
          v-if="$v.form.email.$error && !$v.form.email.required"
          class="error"
        >O email é obrigatório</div>
        <div v-if="$v.form.email.$error && !$v.form.email.email" class="error">O email é inválido</div>
      </div>

      <div v-if="emailCheckedInDB" class="form-group">
        <label class="form-label" for="password">Senha</label>
        <input
          v-model="$v.form.password.$model"
          type="password"
          placeholder="Senha super secreta"
          class="form-control"
          id="password"
        />
        <div
          v-if="$v.form.password.$error && !$v.form.password.required"
          class="error"
        >A senha é obrigatório</div>
        <div
          v-if="$v.form.password.$error && !$v.form.password.correct"
          class="error"
        >Senha inválida tente novamente</div>
      </div>

      <div v-if="existingUser" class="form-group">
        <button @click.prevent="login" class="btn">Logar</button>
      </div>

      <div v-if="emailCheckedInDB && !existingUser" class="form-group">
        <label class="form-label" for="name">Nome</label>
        <input
          v-model="$v.form.name.$model"
          type="text"
          placeholder="Como gosta de ser chamado?"
          class="form-control"
          id="name"
        />
        <div v-if="$v.form.name.$error" class="error">O nome é obrigatório</div>
      </div>
    </form>
    <div v-else class="text-center">
      Logado com sucesso!
      <a href="#" @click="reset">Não é {{form.name}}?</a>
    </div>
  </div>
</template>

<script>
import { authenticateUser, checkIfUserExistsInDB } from "../api";
import { required, email } from "vuelidate/lib/validators";

export default {
  data() {
    return {
      form: {
        email: null,
        password: null,
        name: null
      },
      emailCheckedInDB: false,
      existingUser: false,
      wrongPassword: false
    };
  },
  computed: {
    loggedIn() {
      return this.existingUser && this.form.name;
    }
  },
  validations: {
    form: {
      email: {
        required,
        email
      },
      password: {
        required,
        correct() {
          return !this.wrongPassword;
        }
      },
      name: {
        required
      }
    }
  },
  methods: {
    checkIfUserExists() {
      if (!this.$v.form.email.$invalid) {
        this.$emit("updateAsyncState", "pending");
        return checkIfUserExistsInDB(this.form.email)
          .then(() => {
            // Usuário existe
            this.existingUser = true;
            this.emailCheckedInDB = true;
            this.$emit("updateAsyncState", "success");
          })
          .catch(() => {
            this.existingUser = false;
            this.emailCheckedInDB = true;
            this.$emit("updateAsyncState", "success");
          });
      }
    },

    login() {
      this.wrongPassword = false;
      if (!this.$v.form.password.$invalid) {
        this.$emit("updateAsyncState", "pending");
        return authenticateUser(this.form.email, this.form.password)
          .then(user => {
            this.form.name = user.name;
            this.submit();
            this.$emit("updateAsyncState", "success");
          })
          .catch(() => {
            this.wrongPassword = true;
            this.$emit("updateAsyncState", "success");
          });
      }
    },

    reset() {
      this.form.email = null;
      this.form.password = null;
      this.form.name = null;
      this.emailCheckedInDB = false;
      this.wrongPassword = false;
      this.existingUser = false;
      this.$v.$reset();
    },

    submit() {
      this.$emit("update", {
        data: {
          email: this.form.email,
          password: this.form.password,
          name: this.form.name
        },
        valid: !this.$v.$invalid
      });
    }
  }
};
</script>

<style scoped>
</style>