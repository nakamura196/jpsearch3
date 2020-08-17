<template>
  <div v-if="moreLikeThisData.length > 0">
    <!-- <v-divider class="my-10" /> -->
    <h3 class="mb-4">
      {{ $t('similar_images') }}
    </h3>
    <HorizontalItems :key="itemId" :data="moreLikeThisData" />
  </div>
</template>

<script lang="ts">
import { Prop, Component, Watch, Vue } from 'nuxt-property-decorator'
import HorizontalItems from '~/components/display/HorizontalItems.vue'

@Component({
  components: {
    HorizontalItems,
  },
})
export default class morelikethis extends Vue {
  moreLikeThisData: any[] = []

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
    const itemId = this.itemId
    let arr = []
    const data = this.$store.state.data
    const tmp: any = {}
    for (let i = 0; i < data.length; i++) {
      const id = data[i]._id
      tmp[id] = data[i]
      if (id === itemId && data[i].images) {
        arr = data[i].images
      }
    }

    const moreLikeThisData = []

    for (let i = 0; i < arr.length; i++) {
      moreLikeThisData.push(tmp[arr[i]])
    }

    this.moreLikeThisData = moreLikeThisData
  }
}
</script>
