<template>
  <div>
    <dl>
      <dt class="text-muted">{{ $t('about_iiif') }}</dt>
      <dd class="mt-5">
        <v-tooltip bottom>
          <template v-slot:activator="{ on }">
            <v-btn
              class="mr-4 mb-4"
              icon
              :href="manifest"
              target="_blank"
              variant="link"
              v-on="on"
            >
              <v-img width="30px" :src="manifestIcon" />
            </v-btn>
          </template>
          <span>{{ 'IIIF Manifest' }}</span>
        </v-tooltip>

        <v-tooltip v-for="(obj, index) in viewers" :key="index" bottom>
          <template v-slot:activator="{ on }">
            <v-btn
              class="mr-4 mb-4"
              icon
              target="_blank"
              :href="obj.url + encodeURIComponent(manifest)"
              variant="link"
              v-on="on"
            >
              <v-img width="30px" :src="obj.image" />
            </v-btn>
          </template>
          <span>{{ obj.label }}</span>
        </v-tooltip>
      </dd>
    </dl>
  </div>
</template>

<script lang="ts">
import { Prop, Component, Vue } from 'nuxt-property-decorator'
import axios from 'axios'

@Component
export default class ShareButtons extends Vue {
  @Prop({ required: true })
  manifest!: string

  manifestIcon: string = process.env.BASE_URL + '/img/icons/iiif-logo.svg'

  viewers: { [key: string]: string }[] = [
    {
      image: process.env.BASE_URL + '/img/icons/ia.png',
      url: 'http://www.kanzaki.com/works/2016/pub/image-annotator?u=',
      label: 'Image Annotator',
    },
    {
      image: process.env.BASE_URL + '/img/icons/mirador3.svg',
      url: 'http://mirador.cultural.jp/?manifest=',
      label: 'Mirador 3',
    },
    {
      image: process.env.BASE_URL + '/img/icons/mirador.png',
      url: 'http://da.dl.itc.u-tokyo.ac.jp/mirador/?manifest=',
      label: 'Mirador 2',
    },
    {
      image: process.env.BASE_URL + '/img/icons/uv.jpg',
      url: 'http://universalviewer.io/examples/uv/uv.html#?manifest=',
      label: 'Universal Viewer',
    },
  ]

  created() {
    axios.get(this.manifest).then((response) => {
      const result = response.data
      const self = this
      if (
        result['@context'] !== 'http://www.shared-canvas.org/ns/context.json'
      ) {
        self.viewers.push({
          image: process.env.BASE_URL + '/img/icons/icp-logo.svg',
          url:
            'http://codh.rois.ac.jp/software/iiif-curation-viewer/demo/?manifest=',
          label: 'Curation Viewer',
        })

        if (!self.manifest.includes('https://api.cultural.jp/iiif/')) {
          self.viewers.push({
            image: process.env.BASE_URL + '/img/icons/icp-logo.svg',
            url:
              'http://codh.rois.ac.jp/kuronet/iiif-curation-viewer/?manifest=',
            label: 'くずし字認識ビューア / Kuzushiji Recognition Viewer',
          })
        }
      }
    })

    if (!this.manifest.includes('https://api.cultural.jp/iiif/')) {
      this.viewers.push({
        image: process.env.BASE_URL + '/img/icons/tify-logo.svg',
        url: 'http://tify.sub.uni-goettingen.de/demo.html?manifest=',
        label: 'TIFY',
      })
    }
  }
}
</script>
