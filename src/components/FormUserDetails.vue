<template>
  <div>
    <h1 class="title">Criação de conta</h1>

    <h2 class="subtitle">Crie uma conta ou faça login para solicitar sua assinatura de CoffeeBox</h2>

    <form class="form" @input="submit">
      <div class="form-group">
        <label class="form-label" for="email">Email</label>
        <input
          type="text"
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

      <div class="form-group">
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
      </div>

      <div class="form-group">
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
  </div>
</template>

<script>
import { required, email } from "vuelidate/lib/validators";
export default {
  data() {
    return {
      form: {
        email: null,
        password: null,
        name: null
      }
    };
  },
  validations: {
    form: {
      email: {
        required,
        email
      },
      password: {
        required
      },
      name: {
        required
      }
    }
  },
  methods: {
    submit() {
      if (!this.$v.$invalid) {
        this.$emit("update", {
          email: this.form.email,
          password: this.form.password,
          name: this.form.name
        });
      }
    }
  }
};
</script>

<style scoped>
</style>