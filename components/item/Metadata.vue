<template>
  <div>
    <v-container class="mt-0 mb-0">
      <iframe
        v-if="metadata._source._manifest && dataType == 'sc:Collection'"
        :key="metadata._id"
        class="mt-5"
        :src="
          'https://universalviewer.io/examples/uv/./uv.html#?manifest=' +
          encodeURIComponent(metadata._source._manifest)
        "
        width="100%"
        height="600"
        allowfullscreen
        frameborder="0"
      ></iframe>

      <v-row class="mt-5">
        <v-col cols="12" sm="8">
          <v-row>
            <v-col cols="12" :sm="6">
              <v-img
                contain
                align="center"
                :src="$utils.formatArrayValue(metadata._source._thumbnail)"
                height="300"
                class="grey lighten-2"
            /></v-col>
            <v-col cols="12" :sm="6">
              <h2>
                <b>{{ $utils.formatArrayValue(metadata._source._label) }}</b>
              </h2>

              <div class="mt-5">
                <a
                  v-for="(value, index) in metadata._source._license"
                  :key="index"
                  class="mr-2"
                  :href="value"
                  target="_blank"
                  >{{ metadata._source._license[index] }}
                  <v-icon>mdi-open-in-new</v-icon>
                </a>
              </div>

              <div v-if="false" class="mt-5">
                <v-tooltip bottom>
                  <template v-slot:activator="{ on }">
                    <v-btn
                      v-if="metadata._source._manifest"
                      class="grey lighten-3 ma-1"
                      :href="
                        'http://pocket.cultural.jp' +
                        ($i18n.locale == 'ja' ? '' : '/en') +
                        '/?url=' +
                        $utils.formatArrayValue(
                          metadata._source._manifest
                        ) /* +
                        '&related=' +
                        url*/
                      "
                      target="_blank"
                      v-on="on"
                    >
                      {{ $t('save_to_my_list') }}
                    </v-btn>
                  </template>
                  <span>{{ $t('save_to_iiif_pocket') }}</span>
                </v-tooltip>
              </div>
            </v-col>
          </v-row>
          <template v-if="metadata._source._related">
            <v-btn
              target="_blank"
              :href="$utils.formatArrayValue(metadata._source._related)"
              class="primary ma-1"
            >
              {{ $t('view_full_item') }}
              <v-icon>mdi-open-in-new</v-icon>
            </v-btn>
          </template>

          <v-btn
            v-if="metadata._source._curation"
            target="_blank"
            :href="
              'http://codh.rois.ac.jp/software/iiif-curation-viewer/demo/?curation=' +
              $utils.formatArrayValue(metadata._source._curation) +
              '&pos=' +
              $utils.formatArrayValue(metadata._source._pos)
            "
            class="primary ma-1"
          >
            IIIF Curation Viewer
            <v-icon>mdi-open-in-new</v-icon>
          </v-btn>
        </v-col>

        <v-col cols="12" sm="4">
          <v-sheet
            v-if="metadata._source._manifest"
            class="grey lighten-3 py-5 px-3 py-3"
            style="background-color: #f9f6f0;"
          >
            <IIIFViewers
              :manifest="$utils.formatArrayValue(metadata._source._manifest)"
            />
          </v-sheet>

          <v-sheet
            class="grey lighten-3 py-5 px-3 py-3 mt-4"
            style="background-color: #f9f6f0;"
          >
            <share-buttons
              :title="$utils.formatArrayValue(metadata._source._label)"
              :url="url"
              :image="$utils.formatArrayValue(metadata._source._thumbnail)"
              :manifest="$utils.formatArrayValue(metadata._source._manifest)"
            />
          </v-sheet>
        </v-col>
      </v-row>
    </v-container>

    <v-container>
      <template v-for="(obj, field) in sorted(metadata._source)">
        <dl v-if="!field.startsWith('_')" :key="field" class="row mt-0">
          <dt class="col-sm-3 text-muted">{{ field }}</dt>
          <dd class="col-sm-9">
            <template>
              <div v-for="(value, index) in obj" :key="index">
                <nuxt-link
                  :to="
                    localePath({
                      name: 'search',
                      query: $utils.createFacetQuery([
                        { field: 'fc-' + field, value: value },
                        { field: 'u', value: [$route.query.u] },
                      ]),
                    })
                  "
                  >{{ value }}</nuxt-link
                >
              </div>
            </template>
          </dd>
        </dl>
      </template>
    </v-container>

    <v-container fluid>
      <Entities class="mb-5" :item-id="metadata._id" />
      <MoreLikeThis class="mb-5" :item-id="metadata._id" />

      <SimilarImages class="mb-5" :item-id="metadata._id" />
    </v-container>
  </div>
</template>

<script lang="ts">
import { Prop, Component, Vue } from 'nuxt-property-decorator'
import ShareButtons from '~/components/common/ShareButtons.vue'
import IIIFViewers from '~/components/item/IIIFViewers.vue'
import Entities from '~/components/item/Entities.vue'
import MoreLikeThis from '~/components/item/MoreLikeThis.vue'
import SimilarImages from '~/components/item/SimilarImages.vue'

@Component({
  components: {
    ShareButtons,
    IIIFViewers,
    Entities,
    MoreLikeThis,
    SimilarImages,
  },
})
export default class Metadata extends Vue {
  @Prop({ required: true })
  metadata!: any

  get data() {
    return this.metadata
  }

  url = location.href

  get dataType(): string {
    if (this.$store.state.json) {
      return this.$store.state.json['@type']
    } else {
      return ''
    }
  }

  sorted(source: any) {
    const obj: any = {}
    const keys = Object.keys(source) // .sort()
    for (let i = 0; i < keys.length; i++) {
      const key = keys[i]
      obj[key] = source[key]
    }
    return obj
  }
}
</script>
