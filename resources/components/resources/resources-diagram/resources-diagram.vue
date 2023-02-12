<template>
  <div :class="['resources-diagram', { 'resources-diagram--loading': loading }]">
    <div class="resources-diagram__settings">
      <div class="resources-diagram__select">
        <v-select-list :list="periodList"
          @onSelect="onSelect"></v-select-list>
      </div>
      <div class="resources-diagram__legend">
        <div v-for="rate in ratesGrouped"
          :class="['resources-diagram__rate', `resources-diagram__rate-${rate[0].type}`]">
          <div class="resources-diagram__rate-label">
            <v-checkbox :color="rate[0].color.label"
              v-model="rate[0].show"
              small
              class="resources-diagram__rate-checkbox"
              v-if="rate[0].type === 'heating'"></v-checkbox>
            {{ rate[0].typeLabel }}
          </div>

          <div v-if="(rate.length > 1)"
            v-for="subrate in rate"
            class="resources-diagram__subrate">
            <v-checkbox :color="subrate.color.label"
              v-model="subrate.show"
              small
              class="resources-diagram__rate-checkbox"></v-checkbox>
            {{ subrate.name }}
          </div>
        </div>
      </div>
    </div>
    <div class="resources-diagram__diagram">
      <v-chart ref="chart"
        class="diagram"
        :option="options"
        autoresize></v-chart>
    </div>
  </div>
</template>

<script>
import { computed, ref, onBeforeMount, watch } from 'vue'
import VSelectList from '../../ui/v-select-list/v-select-list.vue'
import VCheckbox from '../../ui/v-checkbox/v-checkbox.vue'
import { useResources } from '../../../store/resources'
import { use } from 'echarts/core';
import { CanvasRenderer } from 'echarts/renderers';
import { BarChart } from 'echarts/charts';
import dayjs from "dayjs";
import weekOfYear from "dayjs/plugin/weekOfYear";
import {
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  GridComponent,
  DataZoomComponent,
} from 'echarts/components';
import VChart from 'vue-echarts';

use([
  GridComponent,
  CanvasRenderer,
  BarChart,
  TitleComponent,
  TooltipComponent,
  LegendComponent,
  DataZoomComponent,
]);

dayjs.extend(weekOfYear)

export default {
  name: 'resources-diagram',
  components: { VSelectList, VCheckbox, VChart },
  emit: ['onSave'],
  props: {
    premise: {
      type: Object,
      required: true,
      default: false,
    },
    loading: {
      type: Boolean,
      required: false,
      default: false,
    },
    types: {
      type: Array,
      required: true,
      default: (() => [])
    }
  },
  setup(props) {
    const storeResources = useResources()
    const { getMeterReadings } = storeResources

    const period = ref('today')
    const data = ref()
    const legend = ref([])
    const chart = ref()
    const loading = ref(false)
    const periodList = ref(
      [
        {
          name: 'за сегодня',
          onSelect: () => {
            period.value = 'today'
          }
        },
        {
          name: 'за неделю',
          onSelect: () => {
            period.value = 'week'
          }
        },
        {
          name: 'за месяц',
          onSelect: () => {
            period.value = 'month'
          }
        },
        {
          name: 'за текущий год',
          onSelect:
            () => {
              period.value = 'year'
            }
        },
        {
          name: 'за последний год',
          onSelect:
            () => {
              period.value = 'last_year'
            }
        },
      ]
    )
    const ratesGrouped = ref({})

    const barOption = {
      type: 'bar',
      barGap: 0.2,
      barCategoryGap: 37,
      cursor: 'default'
    }

    const itemStyle = {
      color: {
        type: 'linear',
        x: 0,
        y: 0,
        x2: 0,
        y2: 1,
      }
    }

    let curMonth = dayjs().format('M')

    const options = computed(() => {
      let option = {
        grid: {
          height: 230,
          top: 0,
          left: 0,
          right: 0,
          bottom: 0
        },
        legend: {
          show: false,
        },
        xAxis: [
          {
            type: 'category',
            axisLine: {
              show: false,
            },
            axisTick: {
              show: false,
            },
          },

        ],
        yAxis: [
          {
            show: false,
            type: 'value'
          }
        ],
        animation: false,
        dataZoom: [
          {
            type: 'inside',
            xAxisIndex: 0,
            start: 0,
            end: 100,
            height: 0,
            right: 0,
            left: 0,
            throttle: 0,
            filterMode: 'empty',
          }
        ],
        media: [
          {
            query: { minWidth: 767 },
            option: {
              dataZoom: [
                {
                  end: 100,
                }
              ]
            }
          },
          {
            query: { maxWidth: 767 },
            option: {
              dataZoom: [
                {
                  end: 33,
                }
              ]
            }
          },
        ]
      }

      let monthNames = ['янв', 'фев', 'мар', 'апр', 'май', 'июн', 'июл', 'авг', 'сен', 'окт', 'ноя', 'дек',]
      if (period.value === 'today') {
        option.xAxis[0].data = ['сегодня']
        option.series = legend.value.map(rate => {
          return getBar(rate)
        })
      }
      else if (period.value === 'week') {
        option.xAxis[0].data = ['пн', 'вт', 'ср', 'чт', 'пт', 'сб', 'вс']
        option.series = legend.value.map(rate => {
          return getBar(rate)
        })
      }
      else if (period.value === 'month') {
        option.xAxis[0].data = ['неделя 1', 'неделя 2', 'неделя 3', 'неделя 4', 'неделя 5']
        option.series = legend.value.map(rate => {
          return getBar(rate)
        })
      }
      else if (period.value === 'year') {
        option.xAxis[0].data = monthNames
        option.series = legend.value.map(rate => {
          return getBar(rate)
        })
      }
      else if (period.value === 'last_year') {
        let secondPart = monthNames.splice(0,curMonth - 1)
        monthNames = monthNames.concat(secondPart)
        option.xAxis[0].data = monthNames
        option.series = legend.value.map(rate => {
          return getBar(rate)
        })
      }
      return option
    })

    // options for series bars
    const getBar = (rate) => {
      let newBar = {
        ...barOption,
        name: rate.name,
        show: rate.show,
        data: rate.show ? rate.data : [],
        itemStyle: {
          ...itemStyle,
          borderRadius: rate.radius,
          color: {
            ...itemStyle.color,
            colorStops: rate.color.value?.map((color, i) => ({ offset: i, color: color })),
          }
        },
      }
      switch (period.value) {
        case 'today':
          newBar.barWidth = rate.show ? '4%' : 1
          break;
        case 'week':
          newBar.barWidth = rate.show ? 10 : 1
          break;
        case 'month':
          newBar.barWidth = rate.show ? 15 : 1
          break;
        case 'year':
          newBar.barWidth = rate.show ? 5 : 1
          break;
        case 'last_year':
          newBar.barWidth = rate.show ? 5 : 1
          break;
      }
      return newBar
    }

    const getColor = (name) => {
      switch (name) {
        case 'Дневной':
          return {
            label: 'orange',
            value: ['#F7B26E', '#F79C6E']
          }
        case 'Ночной':
          return {
            label: 'blue',
            value: ['#6EA7F7', '#6E90F7']
          }
        case 'Пиковый':
          return {
            label: 'purple',
            value: ['#9A6EF7', '#896EF7']
          }
        case 'Холодная':
          return {
            label: 'light-blue',
            value: ['#6ECEF7', '#6EB7F7']
          }
        case 'Горячая':
          return {
            label: 'red',
            value: ['#F7856E', '#F76E6E']
          }
        case 'Основное':
          return {
            label: 'green',
            value: ['#ADD048', '#8CB31A']
          }
      }
    }

    const updateLegend = () => {
      let res = props.types.map(type => (
        type.rates.map(rate => {
          return {
            name: rate.label ? rate.label : rate.name,
						title: rate.name,
            show: true,
            color: getColor(rate.name),
            type: type.name === 'Электроэнергия' ? 'elect' : type.name === 'Водоснабжение' ? 'water' : 'heating',
            typeLabel: type.label && type.name !== 'Водоснабжение' ? type.label : type.name,
            data: []
          }
        })
      ))

      //one array from many arrays
      res = res.reduce((res, array) => {
        res.push(...array);
        return res;
      }, []);

      res.splice(3, 0, {
        name: 'empty',
        color: '#26c238',
        show: true,
      })
      res.splice(6, 0, {
        name: 'empty',
        color: '#26c238',
        show: true,
      })

      legend.value = res
      fillLegendData()
    }

    const fillLegendData = () => {
      const firstDayOfMonth = dayjs(dayjs(data.value[0]?.UF_DATE).date(1)).day()

      const getDataOpacity = (date) => {
        switch (period.value) {
          case 'week':
            return dayjs().isSame(date, 'day') ? 0.5 : 1
          case 'month':
            return dayjs().isSame(date, 'week') ? 0.5 : 1
          case 'year':
            return dayjs().isSame(date, 'month') ? 0.5 : 1
          default:
            return 1
        }
      }

      data.value?.map(row => {
        let curRate
        if (row.rate.data.UF_TITLE === 'Основное') {
          curRate = legend.value.find(rate => rate.name === 'Основное')
        }
        else {
          curRate = legend.value.find(rate => rate.title === row.rate.data.UF_TITLE)
        }
        if (curRate) {
          switch (period.value) {
            case 'today':
              curRate.data[0] = row.UF_DIFFERENCE_READINGS
              curRate.radius = [40, 40, 0, 0]
              curRate.width = '4%'
              break;
            case 'week':
              curRate.data[dayjs(row.UF_DATE).day() - 1] = { value: row.UF_DIFFERENCE_READINGS, itemStyle: { opacity: getDataOpacity(row.UF_DATE) } }
              curRate.radius = [10, 10, 0, 0]
              curRate.width = 10
              break;
            case 'month':
              let day = dayjs(row.UF_DATE).format('DD')
              let week = Math.floor((+day + firstDayOfMonth) / 7)
              curRate.data[week - 1] = { value: row.UF_DIFFERENCE_READINGS, itemStyle: { opacity: getDataOpacity(row.UF_DATE) } }
              curRate.radius = [15, 15, 0, 0]
              curRate.width = 15
              break;
            case 'year':
              curRate.data[dayjs(row.UF_DATE).format('M') - 1] = { value: row.UF_DIFFERENCE_READINGS, itemStyle: { opacity: getDataOpacity(row.UF_DATE) } }
              curRate.radius = [5, 5, 0, 0]
              curRate.width = 5
              break;
            case 'last_year':
              let rowMonth = dayjs(row.UF_DATE).format('M')
              let indexToInsert = curMonth <= rowMonth ? rowMonth - curMonth : 12 - curMonth + rowMonth
              curRate.data[indexToInsert] = { value: row.UF_DIFFERENCE_READINGS, itemStyle: { opacity: getDataOpacity(row.UF_DATE) } }
              curRate.radius = [5, 5, 0, 0]
              curRate.width = 5
              break;
          }

        }
      })
    }

    const groupBy = (array, key) => {
      return array.reduce((result, currentValue) => {
        if (currentValue.type) {

          ((result[currentValue[key]] = result[currentValue[key]] || [])).push(
            currentValue
          );
        }
        loading.value = false
        return result;
      }, {});

    };

    const onSelect = (callback) => {
      callback()
      updateMeterReadings()
    }

    const updateMeterReadings = () => {
      loading.value = true
      getMeterReadings(props.premise.ID, period.value).then(res => {
        data.value = res.data.data
        updateLegend()
        ratesGrouped.value = groupBy(legend.value, 'type')
      })
    }

    onBeforeMount(() => {
      updateMeterReadings()
    })

    watch(() => props.premise, val => {
      updateMeterReadings()
    }, { deep: true })

    watch(() => legend.value, val => {
      if (!val[4]?.show && !val[5]?.show && val[6]) {
        legend.value[6].show = false
      }
      else {
        legend.value[6].show = true
      }
    }, { deep: true })

    return {
      periodList,
      data,
      options,
      chart,
      ratesGrouped,
      legend,
      loading,
      onSelect,
    }
  }
}
</script>

<style lang="scss" src="./resources-diagram.scss" scoped />



