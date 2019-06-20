<template>
  <div class="container">
    <h1> {{ title }} </h1>
    <p class= "lead">
      Página que lista issues de um repositório do Github
    </p>

    <div v-if="response.status === 'error'" class="alert alert-danger">
          {{ response.message }}
    </div>

    <div class="row">
      <div class="col">
        <div class="form-group">
          <input v-model="username" type="text" class="form-control" placeholder="github username">
        </div>
      </div>

      <div class="col">
        <div class="form-group">
          <input v-model="repository" type="text"
                 class="form-control" placeholder="github repositorio">
        </div>
      </div>

      <div class="col-3">
        <div class="form-group">
          <button @click.prevent.stop="getIssues()"
                  class="btn btn-success">GO</button>

          <button @click.prevent.stop="reset()"
                  class="btn btn-warning">LIMPAR</button>
        </div>
      </div>
    </div>
    <br><hr><br>

    <table class="table table-sm table-bordered">
      <thead>
        <tr>
          <th width="100">Número</th>
          <th>Título</th>
        </tr>
      </thead>

      <tbody>
        <tr v-if="loader.getIssues">
          <td class="text-danger" colspan="2"><img src="../../static/loading.svg" alt=""></td>
        </tr>
        <template  v-if="this.showIssues">
          <tr v-for="issue in issues"
              v-bind:key="issue.number">
              <td>

                <router-link v-bind:to="{ name: 'GitHubIssue',
                                          params: {
                                            name: username,
                                            repo: repository,
                                            issue: issue.number,
                                  }}">
                  {{ issue.number }}
                </router-link>

                <!-- <img v-if="loader.getIssues"
                    src="../../static/loading.svg" alt=""> -->

              </td>
              <td>
                {{issue.title}}
              </td>
          </tr>
        </template>

        <tr v-if="noIssues">
          <td class="text-center" colspan="2"> Nenhuma issue encontrada</td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
import axios from 'axios';

export default {
  name: 'GitHubIssues',

  created() {
    this.getLocalData();
  },

  data() {
    return {
      title: 'Vue.Js e Github',
      username: '',
      repository: '',
      issues: [],
      response: {
        status: '',
        message: 'Houve alguma problema na conexão',
      },
      loader: {
        getIssues: false,
      },
    };
  },

  computed: {
    showIssues() {
      return !!this.issues.length && !this.loader.getIssues;
    },
    noIssues() {
      return !this.issues.length && !this.loader.getIssues;
    },
  },

  methods: {
    reset() {
      this.username = '';
      this.repository = '';
      localStorage.removeItem('gitHubIssues');
      this.resetResponse();
    },

    resetResponse() {
      this.response.status = '';
      this.response.message = '';
    },

    getIssues() {
      this.resetResponse();
      this.issues = [];

      if (this.username && this.repository) {
        this.loader.getIssues = true;
        localStorage.setItem('gitHubIssues', JSON.stringify({ username: this.username, repository: this.repository }));
        const url = `https://api.github.com/repos/${this.username}/${this.repository}/issues`;
        axios.get(url).then((response) => {
          this.issues = response.data;
        }).catch(() => {
          this.response.status = 'error';
          this.response.message = 'Repositório não existe';
        }).finally(() => {
          this.loader.getIssues = false;
        });
      }
    },

    getLocalData() {
      const localData = JSON.parse(localStorage.getItem('gitHubIssues'));
      if (localData && localData.username && localData.repository) {
        this.username = localData.username;
        this.repository = localData.repository;
        this.getIssues();
      }
    },
  },
};
</script>
