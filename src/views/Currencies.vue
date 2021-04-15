<template>
  <div class="mx-3 mt-2 mb-3 currencies">
    <h1>Курсы валют ЦБ РФ</h1>

    <template v-if="dailyJson.Valute">
      Последнее обновление базы данных: {{ dailyJsonDate }}<br>
      <template v-if="dailyJson.Valute.USD">
        Доллар США $ — {{ dailyJson.Valute.USD.Value }} руб.
        <span :style="{'color': getColor(dailyJson.Valute.USD.Delta)}">
          {{ getArrow(dailyJson.Valute.USD.Delta) }}
        </span> ({{ dailyJson.Valute.USD.Delta }})<br>
      </template>
      <template v-if="dailyJson.Valute.EUR">
        Евро € — {{ dailyJson.Valute.EUR.Value }} руб.
        <span :style="{'color': getColor(dailyJson.Valute.EUR.Delta)}">
          {{ getArrow(dailyJson.Valute.EUR.Delta) }}
        </span> ({{ dailyJson.Valute.EUR.Delta }})<br>
      </template>
    </template>

    <v-card>
      <v-card-title>
        <v-spacer />
        <v-text-field
          v-model="search"
          append-icon="mdi-magnify"
          label="Поиск"
          single-line
          hide-details
        />
      </v-card-title>
      <v-data-table
        :headers="headers"
        :items="currencies"
        :search="search"
        fixed-header
        disable-pagination
        hide-default-footer
        class="elevation-1"
        :loading="loading"
        loading-text="Данные загружаются..."
        no-data-text="Не удалось загрузить данные"
      >
        <template v-slot:item.Delta="{ item }">
          <span :style="{'color': getColor(item.Delta)}">
            {{ item.Delta }}
          </span>
        </template>
      </v-data-table>
    </v-card>
    <a href="https://www.cbr-xml-daily.ru" target="_new">
      Курсы валют ЦБ РФ в XML
    </a>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'

interface DailyJson {
  Date: string;
  PreviousDate: string;
  PreviousURL: string;
  Timestamp: string;
  Valute: Record<string, Currency>;
}

interface Currency {
  CharCode: string;
  ID: string;
  Name: string;
  Nominal: number;
  NumCode: string;
  Previous: number;
  Value: number;
}

interface TableCurrency extends Currency {
  Delta: string;
}

export default Vue.extend({
  name: 'Currencies',

  data: () => ({
    loading: false,
    error: false,
    search: '',
    dailyJson: {} as DailyJson,
    currencies: [] as TableCurrency[],
    headers: [
      {
        text: 'Цифр. код',
        value: 'NumCode',
        sortable: true,
        align: 'start'
      },
      {
        text: 'Букв. код',
        value: 'CharCode',
        sortable: true
      },
      {
        text: 'Единиц',
        value: 'Nominal',
        sortable: true
      },
      {
        text: 'Валюта',
        value: 'Name',
        sortable: true
      },
      {
        text: 'Курс',
        value: 'Value',
        sortable: true
      },
      {
        text: 'Динамика',
        value: 'Delta',
        sortable: true
      }
    ]
  }),

  computed: {
    dailyJsonDate (): string {
      if (!this.dailyJson.Date) {
        // new Date().toLocaleDateString('ru-RU')
        return '—'
      }
      return new Date(this.dailyJson.Date).toLocaleString('ru-RU')
    }
  },

  mounted () {
    this.getDailyJson()
  },

  methods: {
    getDailyJson (): void {
      this.loading = true
      this.error = false
      fetch('https://www.cbr-xml-daily.ru/daily_json.js')
        .then((response) => {
          return response.json()
        })
        .then((data: DailyJson) => {
          // console.log(data)
          this.dailyJson = data
          Object.entries(data.Valute).forEach(([code, currency]) => {
            const tableCurrency = currency as TableCurrency
            let delta = (currency.Value - currency.Previous).toFixed(4)
            if (parseFloat(delta) > 0) {
              delta = '+' + delta
            }
            tableCurrency.Delta = delta
            this.currencies.push(tableCurrency)
          })
          this.loading = false
        })
        .catch((error) => {
          console.error(error)
          this.loading = false
          this.error = true
        })
    },
    getColor (delta: string) {
      if (delta[0] === '+') return 'green'
      else if (delta[0] === '-') return 'red'
    },
    getArrow (delta: string) {
      if (delta[0] === '+') return '▲'
      else if (delta[0] === '-') return '▼'
    }
  }
})
</script>
