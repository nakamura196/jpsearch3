<template>
  <div>
    <v-sheet color="grey lighten-3">
      <v-container class="py-4">
        <h1>
          {{ $t('properties') }}
        </h1>
      </v-container>
    </v-sheet>

    <v-container>
      <v-card
        v-for="(chartData, index) in chartDataArray"
        :key="index"
        class="my-10"
      >
        <v-card-title class="headline grey lighten-2">
          {{ chartData.property }}
        </v-card-title>

        <v-container>
          <Chart
            :key="index"
            :height="400"
            class="mb-4"
            :chart-data="chartData"
          ></Chart>
        </v-container>
      </v-card>

      <div class="text-center pa-10">
        <v-progress-circular
          v-if="loadingFlag"
          indeterminate
          color="primary"
        ></v-progress-circular>
      </div>
    </v-container>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import axios from 'axios'
import Chart from '~/components/chart/BarChart.vue'

@Component({
  components: {
    Chart,
  },
})
export default class PageProperties extends Vue {
  chartDataArray: any[] = []

  loadingFlag: boolean = true

  async mounted() {
    const properties: string[] = [
      'rdf:type',
      'rdfs:label',

      'schema:contributor',
      'schema:creator',
      'schema:publisher',
      'schema:temporal',
      'schema:dateCreated',
      'schema:datePublished',
      'schema:spatial',
      'schema:about',
      'schema:isbn',
      'schema:issn',
      'schema:inLanguage',
      'schema:image',
      // "schema:description",
      'schema:isPartOf',
      'schema:relatedLink',
      /*
      'jps:agential',
      'jps:temporal',
      'jps:spatial',
      'jps:partOf',
      'jps:relatedLink',
      */
      'jps:accessInfo',
      'jps:sourceInfo',
      'schema:name',
      'schema:identifier',
    ]

    for (let i = 0; i < properties.length; i++) {
      const property = properties[i]
      let sparqlQuery
      if (property.includes('jps')) {
        sparqlQuery =
          `
            prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
            prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
            prefix schema: <http://schema.org/>
            prefix jps: <https://jpsearch.go.jp/term/property#>
            select distinct count(?s) as ?c ?o where {
              ?s ` +
          property +
          ` ?node;
              jps:sourceInfo ?si . 
              ?node schema:provider ?o .
            } order by desc(?c) limit 20
      `
      } else {
        sparqlQuery =
          `
            prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
            prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
            prefix schema: <http://schema.org/>
            prefix jps: <https://jpsearch.go.jp/term/property#>
            select distinct count(?s) as ?c ?o where {
              ?s ` +
          property +
          ` ?o;
              jps:sourceInfo ?si . 
            } order by desc(?c) limit 20
      `
      }

      const url = 'https://jpsearch.go.jp/rdf/sparql?query='
      const self = this

      await axios
        .get(url + encodeURIComponent(sparqlQuery) + '&output=json')
        .then((response: any) => {
          const bindings: any[] = response.data.results.bindings

          const labels: string[] = []
          const dataset: number[] = []

          for (let j = 0; j < bindings.length; j++) {
            const b = bindings[j]
            let o = b.o.value
            if (b.o.type !== 'literal') {
              const tmpSplit = o.split('/')
              o = tmpSplit[tmpSplit.length - 1]
            }
            labels.push(o)
            dataset.push(b.c.value)
          }

          const chartData: any = {
            property,
            labels,
            datasets: [{ data: dataset, label: self.$t('# of items') }],
          }

          self.chartDataArray.push(chartData)
        })
    }

    this.loadingFlag = false
  }

  head() {
    return {
      title: this.$t('properties'),
    }
  }
}
</script>
