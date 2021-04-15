<template>
  <ymap-marker
    :coords="[latitude, longitude]"
    :icon="icon"
    marker-type="placemark"
    marker-id="plane"
    hint-content="Самолётик"
  />
</template>

<script lang="ts">
// eslint-disable-next-line @typescript-eslint/ban-ts-ignore
// @ts-ignore
import { ymapMarker } from 'vue-yandex-maps'
import Vue from 'vue'

export default Vue.extend({
  name: 'Plane',

  components: {
    ymapMarker
  },

  props: {
    latitude: {
      type: Number,
      required: true
    },
    longitude: {
      type: Number,
      required: true
    },
    angle: {
      type: Number,
      default: 0
    },
    width: {
      type: Number,
      default: 60
    },
    height: {
      type: Number,
      default: 60
    },
    img: {
      type: String,
      default: require('../assets/plane.gif')
    }
  },

  computed: {
    icon (): object {
      return {
        layout: 'default#imageWithContent', // 'default#imageWithContent' для использования с контентом
        imageHref: '', // адрес изображения или data:image/svg+xml;base64
        imageSize: [this.width, this.height], // размер иконки в px
        imageOffset: [-this.width / 2, -this.height / 2], // смещение иконки в px,
        /* Следующие поля актуальны для layout: 'default#imageWithContent' */
        content: 'some content here', // содержимое контента
        // contentOffset: [-this.width / 2, -this.height / 2], // смещение контента в px,
        contentLayout: `
          <img
            src="${this.img}"
            style="transform: rotate(${this.angle}deg); width: ${this.width}px; height: ${this.height}px"
            alt="самолётик"
          />
        ` // строковый HTML шаблон для контента
      }
    }
  }
})
</script>
