<template>
  <div style="width: 100%; height: 100%;">
    <yandex-map
      ref="yandexMap"
      :settings="settings"
      :coords="coords"
      @click="onMapClick"
      @map-was-initialized="initMapInstance"
    >
      <plane
        :latitude="planeLatitude"
        :longitude="planeLongitude"
        :angle="planeAngle"
      />
    </yandex-map>

    <v-fab-transition>
      <v-btn
        :key="1"
        color="primary"
        fab
        large
        dark
        bottom
        right
        class="v-btn--example"
        @click="openPlaneDialog"
      >
        <v-icon>mdi-airplane</v-icon>
      </v-btn>
    </v-fab-transition>

    <v-dialog
      v-model="planeDialog"
      width="500"
    >
      <v-card>
        <v-card-title class="headline">
          Настройки самолётика
          <v-spacer />
          <v-btn
            icon
            @click="planeDialog = false"
          >
            <v-icon>mdi-close</v-icon>
          </v-btn>
        </v-card-title>

        <v-divider />

        <v-card-text>
          <v-form
            ref="planeForm"
            v-model="planeFormValid"
            lazy-validation
          >
            <v-text-field
              v-model="planeForm.latitude"
              :rules="planeFormRules.latitude"
              label="Широта"
              required
            />

            <v-text-field
              v-model="planeForm.longitude"
              :rules="planeFormRules.longitude"
              label="Долгота"
              required
            />

            <v-text-field
              v-model="planeForm.speed"
              :rules="planeFormRules.speed"
              label="Скорость"
              required
            />
          </v-form>
        </v-card-text>

        <v-divider />

        <v-card-actions>
          <v-spacer />
          <v-btn
            color="primary"
            text
            @click="submitPlaneForm"
          >
            Сохранить
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </div>
</template>

<script lang="ts">
// eslint-disable-next-line @typescript-eslint/ban-ts-ignore
// @ts-ignore
import { yandexMap } from 'vue-yandex-maps'
import Vue from 'vue'
import Plane from './../components/Plane.vue'

const INITIAL_COORDS = [
  55.12115531674111,
  36.595896285632456
]

export default Vue.extend({
  name: 'Map',

  components: {
    yandexMap,
    Plane
  },

  data: () => ({
    settings: {
      apiKey: '',
      lang: 'ru_RU',
      coordorder: 'latlong',
      version: '2.1'
    },
    coords: INITIAL_COORDS,
    mapInstance: null as any,
    timeouts: [] as number[],
    planeDialog: false,
    planeAngle: 0,
    planeSpeed: 300,
    planeFps: 30,
    planeLatitude: INITIAL_COORDS[0],
    planeLongitude: INITIAL_COORDS[1],
    planeForm: {
      latitude: 0,
      longitude: 0,
      speed: 0
    },
    planeFormRules: {
      latitude: [
        (v: number) => !!v || 'Введите широту',
        (v: number) => (v && v >= -90 && v <= 90) || 'Широта должна быть от -90° до +90°'
      ],
      longitude: [
        (v: number) => !!v || 'Введите долготу',
        (v: number) => (v && v >= -180 && v <= 180) || 'Долгота должна быть от -180° до +180°'
      ],
      speed: [
        (v: number) => !!v || 'Введите скорость',
        (v: number) => (v && v > 0) || 'Скорость должна быть больше 0'
      ]
    },
    planeFormValid: true
  }),

  mounted () {
    console.log(this.$refs.yandexMap)
  },

  destroyed () {
    this.clearTimeouts()
  },

  methods: {
    initMapInstance (mapInstance: any): void {
      this.mapInstance = mapInstance
    },

    getDistance (from: [number, number], to: [number, number]): number {
      return this.mapInstance !== null && this.mapInstance !== undefined &&
      Object.prototype.hasOwnProperty.call(this.mapInstance, '_projection')
        ? this.mapInstance._projection.getCoordSystem().getDistance(from, to)
        : 0
    },

    onMapClick (event: any): void {
      const coords = event.get('coords')
      console.log(coords)
      this.flyToCoords(coords)
    },

    flyToCoords (coords: [number, number]): void {
      const atan = Math.atan2(
        (coords[1] - this.planeLongitude),
        (coords[0] - this.planeLatitude)
      )
      let angle = Number((atan * 180 / Math.PI).toFixed(2))
      if (angle <= 0) {
        angle += 360
      }
      this.planeAngle = angle
      console.log('angle', angle)

      const distance = this.getDistance(
        [this.planeLatitude, this.planeLongitude],
        coords
      )
      console.log('distance', distance)
      // умножаем кол-во секунд на множитель
      const numberOfCoords =
        Math.ceil(distance / this.planeSpeed * this.planeFps)
      const latStep =
        (coords[0] - this.planeLatitude) / (numberOfCoords + 1)
      const lonStep =
        (coords[1] - this.planeLongitude) / (numberOfCoords + 1)
      const interpolatedCoords: Array<[number, number]> = []

      for (let i = 1; i <= numberOfCoords; i++) {
        interpolatedCoords.push([
          this.planeLatitude + latStep * i,
          this.planeLongitude + lonStep * i
        ])
      }

      this.timeouts.forEach(timeout => {
        clearTimeout(timeout)
      })
      // todo: проверить и переписать на использование let timeout вместо массива,
      // а также попробовать вложенный таймаут: https://learn.javascript.ru/settimeout-setinterval
      interpolatedCoords.forEach((coords, index) => {
        const timeout = setTimeout(() => {
          this.planeLatitude = coords[0]
          this.planeLongitude = coords[1]
        }, index * 1000 / this.planeFps) // а тут делим секунду на множитель
        this.timeouts.push(timeout)
      })
    },

    clearTimeouts (): void {
      this.timeouts.forEach(timeout => {
        clearTimeout(timeout)
      })
    },

    openPlaneDialog (): void {
      this.planeForm = {
        latitude: this.planeLatitude,
        longitude: this.planeLongitude,
        speed: this.planeSpeed
      }
      this.planeDialog = true
    },

    submitPlaneForm (): void {
      // if (!this.$refs.planeForm.validate()) {
      if (!this.planeFormValid) {
        return
      }
      this.planeLatitude = this.planeForm.latitude
      this.planeLongitude = this.planeForm.longitude
      this.planeSpeed = this.planeForm.speed
      this.planeDialog = false
    }
  }
})
</script>

<style scoped>
  .ymap-container {
    width: 100%;
    height: 100%;
  }

  /* This is for documentation purposes and will not be needed in your application */
  .v-btn--example {
    bottom: 0;
    position: absolute;
    margin: 0 16px 16px 0;
    right: 0;
  }
</style>
