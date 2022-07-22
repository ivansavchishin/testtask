<template>
  <div id="app">
    <v-app>
      <v-main>
        <v-container>
          <v-select
            v-model="value.name"
            :items="coins"
            item-title="short"
            item-value="short"
            attach
            chips
            label="Cripto"
            multiple
          ></v-select>

          <v-row>
            <v-col v-for="coin in filteredCoins" :key="coin.short" cols="4">
              <v-card>
                <v-row class="">
                  <v-col class="text-h3" xs="12" sm="8" md="8" lg="8"> {{ coin.short }}</v-col>
                  <v-col xs="12" sm="4" md="4" lg="4">
                    <v-img
                      class=""
                      width="50"
                      :aspect-ratio="1"
                      :src="`https://www.cryptocompare.com${coin.icon}`"
                    ></v-img>
                  </v-col>
                </v-row>

                <v-card-title>{{ coin.price }}</v-card-title>
                <v-card-subtitle>{{ coin.name }} </v-card-subtitle>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-main>
    </v-app>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref, computed, reactive, ComputedRef, watch } from 'vue'
import axios from 'axios'

interface Coin {
  short: string
  name: string
  icon: string
  price: number
}

interface CoinInfo {
  Name: string
  FullName: string
  ImageUrl: string
}
interface CurrencyInfo {
  PRICE: number
}

interface RAW {
  USD: CurrencyInfo
}

//generic
// TODO #1: Response Interface
interface CoinResponse {
  CoinInfo: CoinInfo
  RAW?: RAW
}

interface Response {
  Data: CoinResponse[]
}

const coins = ref<Coin[]>([])

const value = reactive({
  name: ['BTC', 'SOL'],
})

const filteredCoins: ComputedRef<Coin[]> = computed(() => {
  return coins.value.filter((coin) => value.name.includes(coin.short))
})

watch(value, (newValue) => {
  console.log(newValue.name.toString())
  localStorage.setItem('names', JSON.stringify(newValue.name))
})

// function filterСoins() {
//   coins1.value = coins.value.filter(function (el) {
//     for (let i = 0; i < value.name.length; i++) {
//       if (el.short === value.name[i]) {
//         return el.short
//       }
//     }
//   })
// }

window.addEventListener('storage', (event) => {
  if (!event.newValue) return
  value.name = JSON.parse(event.newValue)
})

// const addCOins = computed(() => {
//   localStorage.setItem('names', JSON.stringify(value.name))

//   console.log(value.name)
//   console.log(filterСoins())
//   filterСoins()
//   return value.name.length > 0 ? localStorage.getItem('names') : 'No'
// })

function parseDataToCoin(Data: CoinResponse): Coin {
  return {
    icon: Data.CoinInfo.ImageUrl,
    name: Data.CoinInfo.FullName,
    price: Data.RAW?.USD.PRICE ?? 0,
    short: Data.CoinInfo.Name,
  }
}

onMounted(async () => {
  const response = await axios.get<Response>(
    'https://min-api.cryptocompare.com/data/top/totalvolfull?limit=100&tsym=USD'
  )
  coins.value = response.data.Data.map(parseDataToCoin)

  return response.data.Data.map(parseDataToCoin)

  /*
  axios
    .get<Response>('https://min-api.cryptocompare.com/data/top/totalvolfull?limit=10&tsym=USD')

    .then((response) => {
      //перероьити на await , зберігати в локал сторідж
      //якщо строка пуста , вивести перші 3
      //gthtl map зробити фільтр по вибраним
      //coins.value = response.data.Data.map(parseDataToCoin)
    })*/
})
</script>
<style src="@vueform/multiselect/themes/default.css"></style>
