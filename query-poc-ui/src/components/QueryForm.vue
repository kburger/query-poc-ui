<template>
  <div>
    <div id="query-form">
      <textarea v-model="sparql"></textarea>
    </div>
    <!-- controls -->
    <div>
      <button @click="query()" :disabled="queryBtnDisabled">Query</button>
    </div>

    <!-- results -->
    <div>
      <table>
        <thead>
          <th v-for="variable in vars" :key="variable">{{variable}}</th>
        </thead>
        <tbody>
          <tr v-for="binding in bindings" :key="binding">
            <td v-for="variable in vars" :key="variable">{{binding[variable].value}}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator';
import axios from 'axios';

const http = axios.create({
  headers: {
    'Accept': 'application/sparql-results+json'
  }
});

@Component
export default class QueryForm extends Vue {
  private sparql = `PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
SELECT * WHERE {
  ?country a <http://dbpedia.org/class/yago/WikicatMemberStatesOfTheEuropeanUnion> .
  ?country <http://www.w3.org/2002/07/owl#sameAs> ?countrySameAs .
  ?countrySameAs wdt:P2131 ?gdp .
}`;
  private queryBtnDisabled = false;
  private vars: Array<String> = [];
  private bindings: Array<any> = [];

  query(): void {
    this.queryBtnDisabled = true;

    http.post('http://localhost:8080', {
      query: this.sparql,
      endpoints: [
        'http://dbpedia.org/sparql',
        'https://query.wikidata.org/sparql'
      ]
    }).then((response) => {
      this.vars = response.data.head.vars;
      this.bindings = response.data.results.bindings;
    }).finally(() => this.queryBtnDisabled = false);
  }
}
</script>

<style scoped>
textarea {
  width: 100%;
  height: 200px;
}
</style>