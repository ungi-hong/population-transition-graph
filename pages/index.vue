<template>
  <div class="container">
    <section class="prefectures-check-box">
      <h2 class="title-text">都道府県</h2>
      <div class="input-content">
        <p
          v-for="item in prefecturesList"
          :key="item.prefCode"
          class="input-item"
        >
          <label>
            <input
              :key="item.prefCode"
              v-model="checkedPrefecturesItems"
              type="checkbox"
              :name="item.prefName"
              :value="item.prefCode"
              @change="onChange"
            />
            <span>{{ item.prefName }}</span>
          </label>
        </p>
      </div>
    </section>
    <section class="highcharts">
      <h2 class="title-text">都道府県別の総人口推移グラフ</h2>
      <highcharts :options="hogeChartOptions"></highcharts>
    </section>
  </div>
</template>

<script lang="ts">
import {
  defineComponent,
  useFetch,
  reactive,
  useContext,
  toRefs,
} from 'nuxt-composition-api'
import { Chart } from 'highcharts-vue'

export default defineComponent({
  components: {
    highcharts: Chart,
  },
  head: {
    title: '都道府県別の総人口推移グラフ',
  },
  setup(_props) {
    const { state, onChange } = useHighCharts()

    return { ...toRefs(state), onChange }
  },
})

// 型の定義

type HogeChartOptions = {
  title: { text: string }
  xAxis: { categories: number[]; crosshair: boolean }
  yAxis: { title: { text: string }; labels: { format: string } }
  series: {}[]
}

type State = {
  prefecturesList: []
  checkedPrefecturesItems: any[]
  hogeChartOptions: HogeChartOptions
}

type CorrectedTotalPopulationData = {
  y: number
  name: number
}

type SeriesData = {
  name: string | null
  data: CorrectedTotalPopulationData
}

const useHighCharts = () => {
  const { app } = useContext()

  // create Data

  const state: State = reactive({
    prefecturesList: [],
    checkedPrefecturesItems: [],

    hogeChartOptions: {
      title: {
        text: '',
      },
      xAxis: {
        categories: [
          1960,
          1965,
          1970,
          1975,
          1980,
          1985,
          1990,
          1995,
          2000,
          2005,
          2010,
          2015,
          2020,
          2025,
          2030,
          2035,
          2040,
          2045,
        ],
        crosshair: true, // マウスを当てると線が出て、数値が分かりやすくなる
      },
      yAxis: {
        title: {
          text: '人口(千人)',
        },
        labels: {
          format: '{value}', // y軸の目盛り幅が値によって動的に変わる
        },
      },
      series: [{ name: '' }],
    },
  })

  // life sycle

  useFetch(async () => {
    const res = await app.$axios.get('api/v1/prefectures', {
      method: 'GET',
      headers: {
        'X-API-KEY': 'ujqwj0KFo37MEjxr1OTyYWkvBp5c8MJxC99SsquU',
      },
    })

    state.prefecturesList = res.data.result
  })

  // create methods

  const onChange = async (event: Event) => {
    if (event.target instanceof HTMLInputElement) {
      const { checked, value } = event.target
      const valueName = event.target.getAttribute('name')

      if (checked) {
        const res = await app.$axios.get(
          `api/v1/population/composition/perYear?prefCode=${value}`,
          {
            method: 'GET',
            headers: {
              'X-API-KEY': 'ujqwj0KFo37MEjxr1OTyYWkvBp5c8MJxC99SsquU',
            },
          }
        )

        const totalPopulationData = res.data.result.data[0]

        const correctedTotalPopulationData: CorrectedTotalPopulationData = totalPopulationData.data.map(
          ({ year, value }: { year: number; value: number }) => {
            return {
              name: year,
              y: Number(String(value).slice(0, -4)), // 桁が長いので千人単位にする
            }
          }
        )

        const seriesData: SeriesData = {
          name: valueName,
          data: correctedTotalPopulationData,
        }

        state.hogeChartOptions.series.push(seriesData)
      } else {
        const seriesNames = state.hogeChartOptions.series.map((item: any) => {
          return item.name
        })

        const correctedSeries = state.hogeChartOptions.series.filter(
          (_item, index: number) => {
            return seriesNames.indexOf(valueName) !== index
          }
        )

        state.hogeChartOptions.series = correctedSeries
      }
    }
  }

  return { state, onChange }
}
</script>

<style lang="scss" scoped>
.container {
  margin-top: 80px;

  .prefectures-check-box {
    max-width: 1000px;
    margin: 0 auto;

    .input-content {
      display: flex;
      flex-wrap: wrap;

      .input-item {
        margin: 0 10px 10px 0;
      }
    }
  }

  .highcharts {
    max-width: 1000px;
    margin: 80px auto 0;
  }

  .title-text {
    font-weight: bold;
    font-size: 18px;
    margin: 0 0 23px 0;
  }
}
</style>
