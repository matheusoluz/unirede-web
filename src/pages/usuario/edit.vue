<template>
  <q-page class="row flex-center fit q-pa-md bg-dark">
    <q-card class="col-xl-6 col-lg-9 col-md-12 col-sm-12 col-xs-12 bg-white">
      <q-card-title class="bg-red text-white q-py-sm">
        {{isNotNew ? 'Editar' : 'Novo'}} Usuário
      </q-card-title>
      <q-card-main class="row q-mx-sm">
        <q-field
          class="col-6 q-px-md q-pt-lg"
          label="Nome"
          label-width="12"
          :error="$v.form.username.$error"
          error-label="Campo obrigatório."
        >
          <q-input
            v-model="form.username"
            color="light"
          />
        </q-field>
        <q-field
          class="col-6 q-px-md q-pt-lg"
          label="Empresa"
          label-width="12"
        >
          <q-input
            v-model="form.realm"
            color="light"
          />
        </q-field>
        <q-field
          class="col-6 q-px-md q-pt-lg"
          label="Email"
          label-width="12"
          :error="$v.form.email.$error"
          error-label="Campo obrigatório."
        >
          <q-input
            v-model="form.email"
            type="email"
            color="light"
          />
        </q-field>
        <q-field
          class="col-6 q-px-md q-pt-lg"
          label="Permissão"
          :error="$v.form.userACL.$error"
          label-width="12"
          error-label="Campo obrigatório."
        >
          <q-select
            v-model="form.userACL"
            placeholder="Tipos de permissões"
            :options="lstRoles"
            popup-max-height="300px"
            color="light"
          />
        </q-field>
        <q-field
          class="col-6 q-px-md q-pt-lg"
          label="Senha"
          label-width="12"
          :error="$v.form.password.$error"
          :error-label="$v.form.password.required ?
          'Campo obrigatório.' :
          $v.form.password.minLength ?
          `A senha deve ter pelo menos ${$v.form.password.$params.minLength.min} caracteres` :
          $v.form.password.maxLength ?
          `A senha deve no máximo ${$v.form.password.$params.maxLength.min} caracteres` :
          'Erro no campo.'"
        >
          <q-input
            v-model="form.password"
            type="password"
            color="light"
          />
        </q-field>
        <q-field
          class="col-6 q-px-md q-pt-lg"
          label="Confirmar senha"
          label-width="12"
          :error="$v.form.repeatPassword.$error"
          error-label="Senhas diferentes"
        >
          <q-input
            v-model="form.repeatPassword"
            type="password"
            color="light"
          />
        </q-field>
      </q-card-main>
      <q-card-actions
        class="q-mx-md q-mb-lg"
        align="center"
      >
        <q-btn
          class="q-mx-md"
          label="Voltar"
          icon="arrow_back_ios"
          color="red"
          size="form-inverted"
          v-close-overlay
        />
        <q-btn
          class="q-mx-md"
          label="Salvar"
          icon="save"
          color="red"
          size="form-inverted"
          @click="Save"
        />
      </q-card-actions>
    </q-card>
  </q-page>
</template>

<script>
import { AxiosCatchMixin } from '../../mixins/AxiosCatch'
import { required, minLength, maxLength, email, sameAs } from 'vuelidate/lib/validators'
const touchMap = new WeakMap()
export default {
  name: 'EditUser',
  props: ['userId'],
  mixins: [AxiosCatchMixin],
  data () {
    return {
      isNotNew: !!Number(this.userId),
      form: {
        id: Number(this.userId),
        username: null,
        email: null,
        realm: null,
        password: null,
        repeatPassword: null,
        userACL: 3
      },
      lstRoles: []
    }
  },
  validations () {
    const validate = {
      form: {
        username: {
          required,
          minLength: minLength(3),
          maxLength: maxLength(30)
        },
        email: {
          required,
          email
        },
        userACL: {
          required
        },
        password: {},
        repeatPassword: {}
      }
    }

    if (!this.isNotNew) {
      validate.form.password = {
        required,
        minLength: minLength(6),
        maxLength: maxLength(30)
      }

      validate.form.repeatPassword = {
        sameAsPassword: sameAs('password')
      }
    }
    return validate
  },
  mounted () {
    this.GetRoles()
    this.Load()
  },
  methods: {
    delayTouch ($v) {
      $v.$reset()
      if (touchMap.has($v)) clearTimeout(touchMap.get($v))
      touchMap.set($v, setTimeout($v.$touch, 1000))
    },
    GetRoles () {
      this.$axios.get('/Roles')
        .then(Res => {
          Res.data.forEach(role => {
            this.lstRoles.push({
              label: role.name,
              sublabel: role.description,
              value: role.id
            })
          })
        })
        .catch(this.AxiosCatchMixin)
    },
    Load () {
      if (this.isNotNew) {
        this.$axios.get(`/Usuarios/${this.form.id}`)
          .then(Res => {
            this.form = Res.data
            this.form.userACL = Res.data.userACL.value
          })
          .catch(this.AxiosCatchMixin)
      }
    },
    Save () {
      this.$v.form.$touch()

      if (this.$v.form.$error) {
        this.$q.notify('O Formulário possui erros. Verifique os dados Informados.')
      } else {
        let form = JSON.parse(JSON.stringify(this.form))
        const formSave = this.$axios.post(`/Usuarios/CreateOrUpdateWhithACL`, { form: form })

        formSave
          .then(Res => {
            this.$q.notify({
              type: 'positive',
              message: 'Cadastro realizado com sucesso!'
            })
            setInterval(() => {
              window.location.reload()
            }, 2000)
          })
          .catch(this.AxiosCatchMixin)
      }
    }
  }
}
</script>
