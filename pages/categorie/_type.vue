<template>
  <div>
    <v-container fluid>
      <v-row :key="imgIndex" dense>
        <v-col
          v-for="(card, cardIndex) in items"
          :key="cardIndex"
          cols="2"
          sm="3"
          md="3"
          class="px-5 pt-3"
        >
          <v-card
            elevation="2"
            class="d-flex flex-column"
            rounded="md"
            max-width="300"
            height="350"
            @click="addCart(card)"
          >
            <v-img :src="cardImages[card.id]" height="60%" max-width="300" class="rounded-t-md" />
            <div class="d-flex justify-end mr-4 mt-n4">
              <v-btn elevation="0" color="green" style="pointer-events: none">
                <div class="text-white">
                  <v-card-title class="pa-0 text-h6" v-text="card.price + ' €'" />
                </div>
              </v-btn>
            </div>
            <v-card-text class="px-1 py-0">
              <v-card-title class="text-h6 py-0" v-text="card.name && card.name.length > 20 ? card.name.substring(0, 20) + '...' : card.name" />
              <v-card-text class="pb-4" v-text="card.description && card.description.length > 60 ? card.description.substring(0, 100) + '...' : card.description" />
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </div>
</template>

<script>
import { mapGetters, mapMutations } from 'vuex'

export default {
  name: 'CategorieType',
  data () {
    return {
      items: [],
      restaurantId: null,
      type: null,
      cardImages: {},
      imgIndex: 0
    }
  },
  async mounted () {
    this.restaurantId = this.$auth.user.restaurant_id
    this.type = this.$route.params.type
    await this.getProducts()
  },

  methods: {
    ...mapMutations(['pushItem']),
    ...mapGetters(['get']),

    async getProducts () {
      const res = await this.$axios({
        method: 'get',
        url: `/api/plat/read/restaurant/${this.restaurantId}/type/${this.type}`
      })

      if (res.data.values) {
        this.items = await res.data.values.map((item) => {
          return {
            ...item
          }
        })
        this.items.forEach(async (item) => {
          this.cardImages[item.id] = await this.fetchImageData(item)
          this.imgIndex++
        })
      }
    },

    addCart (item) {
      this.pushItem({
        key: 'card',
        item
      })
    },

    async fetchImageData (card) {
      try {
        const response = await this.$axios({
          method: 'get',
          url: `/api/image?url=${encodeURIComponent(card.image.url)}`,
          responseType: 'arraybuffer'
        })
        // Convert the binary data to base64 to use in the image src
        const base64Image = Buffer.from(response.data, 'binary').toString('base64')

        // Return the base64 URL of the image
        return `data:${response.headers['content-type']};base64,${base64Image}`
      } catch (error) {
        return null
      }
    }
  }
}
</script>
