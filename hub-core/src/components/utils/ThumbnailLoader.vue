<template>
  <div>
    <img
      :src="src"
      :style="{
        'max-width': '100%',
        'width': width
      }"
      v-if="src"
    >
    <template v-else>
      <gc-icon icon="photo" />
    </template>
  </div>
</template>

<script>
import { find } from 'lodash'

export default {
  props: {
    size: {
      type: String,
      default: 'thumbnail'
    },
    width: {
      type: String,
      default: '100px'
    },
    asset: {
      type: Object,
      default: null
    }
  },
  computed: {
    transforms () {
      return this.asset.transforms ? this.asset.transforms.data : null
    },
    hasTransforms () {
      return this.transforms && this.transforms.length
    },
    src () {
      let url;

      if (this.asset) {
        if (this.asset.thumbnail) {
          return this.asset.thumbnail
        }
        if (this.hasTransforms) {
          const transform = find(this.transforms, (transform) => {
            return transform.handle === this.size
          })
          if (transform) {
            url = transform.url
          }
        } else {
          url = this.asset.url
        }
      }

      if (!url) {
        return null;
      }

      return url
    }
  }
}
</script>
