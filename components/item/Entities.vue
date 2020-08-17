<template>
  <div v-if="result.length > 0">
    <div v-for="(obj, index) in result" :key="index" cols="12" sm="4">
      <h3 class="my-4">{{ obj.property }}</h3>
      <HorizontalEntities :key="obj.property" :data="obj.children" />
    </div>
  </div>
</template>

<script lang="ts">
import { Prop, Component, Watch, Vue } from 'nuxt-property-decorator'
import axios from 'axios'
import HorizontalEntities from '~/components/display/HorizontalEntities.vue'

@Component({
  components: {
    HorizontalEntities,
  },
})
export default class morelikethis extends Vue {
  children: any[] = []

  @Prop({
    required: true,
  })
  itemId!: string

  mounted() {
    this.moreLikeThis()
  }

  @Watch('itemId')
  watchId(): void {
    this.moreLikeThis()
  }

  moreLikeThis() {
    this.result = []
    const itemId = this.itemId

    const data = this.$store.state.data
    for (let i = 0; i < data.length; i++) {
      const id = data[i]._id
      if (id === itemId) {
        const entity = data[i].entity
        if (entity['schema:subject']) {
          const uris = entity['schema:subject']
          this.api(this.$t('entity'), Object.keys(uris))
        }
      }
    }
  }

  result: any = []

  api(property: any, uris: string[]) {
    const endpoint: string = this.$store.state.api
    const self = this
    axios.get(endpoint).then(function (data) {
      const result = data.data
      const map: any = {}

      for (let i = 0; i < result.length; i++) {
        const obj = result[i]
        // const values = obj[property]
        map[obj['@id']] = obj
      }

      const children: any[] = []
      for (let j = 0; j < uris.length; j++) {
        const id = uris[j]
        const obj = map[id]

        if (!obj) {
          continue
        }

        const newObj: any = {
          _id: obj['@id'],
          _source: {
            _label: [obj['rdfs:label'][0]['@value']],
          },
        }

        if (obj['schema:image']) {
          newObj._source._thumbnail = [obj['schema:image'][0]['@id']]
        }

        if (obj['schema:description']) {
          newObj._source._description = [obj['schema:description'][0]['@value']]
        }
        children.push(newObj)
      }

      const parent = map[uris[0]]

      self.result.push({
        parent,
        property,
        children,
      })
    })
  }
}
</script>
