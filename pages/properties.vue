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
            @clickChart="selectedLabelUpdated"
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

    <v-dialog v-model="dialogFlag" width="500">
      <v-card>
        <v-card-title class="headline grey lighten-2">
          {{ selectedLabel.property }} - {{ selectedLabel.label }} -
          {{ pieChartData.total }}
        </v-card-title>

        <v-container>
          <PieChart
            :height="400"
            class="mb-4"
            :chart-data="pieChartData"
          ></PieChart>
        </v-container>

        <v-divider></v-divider>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" @click="search()">
            {{ $t('search') }}
          </v-btn>
          <v-btn @click="dialogFlag = false">
            {{ $t('close') }}
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import axios from 'axios'
import Chart from '~/components/chart/BarChart.vue'
import PieChart from '~/components/chart/PieChart.vue'

@Component({
  components: {
    Chart,
    PieChart,
  },
})
export default class PageProperties extends Vue {
  chartDataArray: any[] = []

  pieChartData: any = {}

  loadingFlag: boolean = true
  dialogFlag: boolean = false

  selectedLabel: any = {}

  objctTypeUriProperties: string[] = []

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
      'schema:isPartOf',
      'schema:relatedLink',
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

          const objctTypeUriProperties = self.objctTypeUriProperties

          for (let j = 0; j < bindings.length; j++) {
            const b = bindings[j]
            const o = b.o.value

            const c = Number(b.c.value)

            if (b.o.type !== 'literal') {
              if (!objctTypeUriProperties.includes(property)) {
                objctTypeUriProperties.push(property)
              }
              // const tmpSplit = o.split('/')
              // o = tmpSplit[tmpSplit.length - 1]
            }
            labels.push(o)
            dataset.push(c)
          }

          self.objctTypeUriProperties = objctTypeUriProperties

          const chartData: any = {
            property,
            labels,
            terms: labels,
            datasets: [
              {
                data: dataset,
                label: self.$t('# of items'),
              },
            ],
          }

          self.chartDataArray.push(chartData)
        })
    }

    this.loadingFlag = false
  }

  selectedLabelUpdated(data: any) {
    this.selectedLabel = data
    this.dialogFlag = true
    this.getDbs(data.property, data.label)
  }

  getColor() {
    return '#' + (((1 << 24) * (Math.random() + 1)) | 0).toString(16).substr(1)
  }

  getDbs(property: string, label: string) {
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
              ?si schema:provider ?o .
              ?node schema:provider ?o2 .
              filter (?o2 = ` +
        (this.objctTypeUriProperties.includes(property)
          ? '<' + label + '>'
          : "'" + label + "'") +
        `)
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
        ` ?o2;
              jps:sourceInfo ?si . 
              ?si schema:provider ?o .
              filter (?o2 = ` +
        (this.objctTypeUriProperties.includes(property)
          ? '<' + label + '>'
          : "'" + label + "'") +
        `)
            } order by desc(?c) limit 20
      `
    }

    const url = 'https://jpsearch.go.jp/rdf/sparql?query='

    const self = this
    axios
      .get(url + encodeURIComponent(sparqlQuery) + '&output=json')
      .then((response: any) => {
        const bindings: any[] = response.data.results.bindings

        const labels: string[] = []
        const dataset: number[] = []
        const backgroundColor: string[] = []
        let total: number = 0

        for (let j = 0; j < bindings.length; j++) {
          const b = bindings[j]
          const o = b.o.value

          const c = Number(b.c.value)

          labels.push(o)
          dataset.push(c)
          backgroundColor.push(this.getColor())
          total += Number(c)
        }

        const chartData: any = {
          property,
          labels,
          terms: labels,
          total: total.toLocaleString(),
          datasets: [
            {
              data: dataset,
              backgroundColor,
              label: self.$t('# of items'),
            },
          ],
        }

        self.pieChartData = chartData
      })
  }

  search() {
    const selectedLabel = this.selectedLabel
    const property = selectedLabel.property
    const label = selectedLabel.label
    let sparqlQuery
    if (property.includes('jps')) {
      sparqlQuery =
        `
            prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
            prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
            prefix schema: <http://schema.org/>
            prefix jps: <https://jpsearch.go.jp/term/property#>
            select distinct ?cho ?label ?image where {
              ?cho ` +
        property +
        ` ?node;
              jps:sourceInfo ?si;
              rdfs:label ?label . 
              OPTIONAL {?cho schema:image ?image}
              ?si schema:provider ?o .
              ?node schema:provider ?o2 .
              filter (?o2 = ` +
        (this.objctTypeUriProperties.includes(property)
          ? '<' + label + '>'
          : "'" + label + "'") +
        `)
            }
      `
    } else {
      sparqlQuery =
        `
            prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
            prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#>
            prefix schema: <http://schema.org/>
            prefix jps: <https://jpsearch.go.jp/term/property#>
            select distinct ?cho ?label ?image where {
              ?cho ` +
        property +
        ` ?o2;
              jps:sourceInfo ?si;
              rdfs:label ?label .
              OPTIONAL {?cho schema:image ?image}
              ?si schema:provider ?o .
              filter (?o2 = ` +
        (this.objctTypeUriProperties.includes(property)
          ? '<' + label + '>'
          : "'" + label + "'") +
        `)
            }
      `
    }

    let url = 'https://jpsearch.go.jp/rdf/sparql/easy/?query='

    url = url + encodeURIComponent(sparqlQuery)
    window.open(url, '_blank')
  }

  head() {
    return {
      title: this.$t('properties'),
    }
  }
}
</script>
