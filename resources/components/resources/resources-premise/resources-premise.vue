<template>
  <div class="resources__premise"
    v-if="counters.length">
    <div class="resources__premise-title">{{ getPremiseTitle(premise) }}</div>
    <div class="resources__table">
      <div class="resources__headers">
        <div class="resources__header">
          <div>{{ getLocalAddress(premise) + ',' }}</div>
          <div>{{ getCityAddress(premise) }}</div>
        </div>
        <div class="resources__header">Текущие показания, дата синхронизации</div>
        <div class="resources__header">Предыдущий месяц</div>
        <div class="resources__header">Дата последнего показания</div>
      </div>
      <div class="resources__types">
        <template v-for="resource in premiseRef">
          <div class="resources__type resources__type-elect"
            v-if="resource.name === 'Электроэнергия'">
            <div class="resources__type-title">{{ resource.label }}
              <svg @click="$emit('onEditCounter', resource)"
                class="resources__type-edit-svg"
                v-svg
                size="10 10"
                symbol="edit"></svg>
            </div>
            <div class="resources__type-info">
              <div class="resources__counter">Счетчик №{{ resource.serialNumber }}
              </div>
              <div class="resources__virification-date">Поверка {{
                resource.checkDate
              }}</div>
            </div>
            <div class="resources__type-row"
              v-for="rate in resource.rates">
              <div class="resources__type-column-name resources__type-column">
                <div class="resources__type-dimension">
                  <div class="resources__type-square"
                    :style="{ background: rate.color }"></div>
                  <div class="resources__type-name">{{ rate.name }}</div>
                </div>
              </div>
              <div class="resources__type-current resources__type-column">
                <div class="resources__header resources__header--mobile">Текущие
                  показания, дата синхронизации</div>
                <div class="resources__type-value">
                  <span v-html="rate.curVal"></span>
                  <span class="resources__type-changes">{{
                    rate.changes
                  }}</span>
                </div>
                <div class="resources__type-date">{{ rate.date }}</div>
              </div>
              <div class="resources__type-previous resources__type-column">
                <div class="resources__header resources__header--mobile">
                  Предыдущий месяц</div>
                <div class="resources__type-value"
                  v-html="rate.prevVal">
                </div>
              </div>
              <div class="resources__type-month resources__type-column">
                <div class="resources__header resources__header--mobile">Дата
                  последнего показания</div>
                <div class="resources__type-value">{{ rate.lastDate }}</div>
              </div>
            </div>
          </div>
          <div class="resources__type resources__type-water"
            v-if="resource.name === 'Водоснабжение'">
            <div class="resources__type-title">{{ resource.name }}</div>
            <div class="resources__type-row"
              v-for="rate in resource.rates">
              <div class="resources__type-column-name resources__type-column">
                <div class="resources__type-dimension">
                  <div class="resources__type-square"
                    :style="{ background: rate.color }"></div>
                  <div class="resources__type-name">{{ rate.label }}
                    <svg @click="$emit('onEditCounter', rate)"
                      class="resources__type-edit-svg"
                      v-svg
                      size="10 10"
                      symbol="edit"></svg>
                  </div>
                </div>
                <div class="resources__type-info">
                  <div class="resources__counter">Счетчик №{{
                    rate.serialNumber
                  }}</div>
                  <div class="resources__virification-date">Поверка {{
                    rate.checkDate
                  }}</div>
                </div>
              </div>
              <div class="resources__type-current resources__type-column">
                <div class="resources__header resources__header--mobile">Текущие
                  показания, дата синхронизации</div>
                <div class="resources__type-value">
                  <span v-html="rate.curVal"></span>
                  <span class="resources__type-changes">{{
                    rate.changes
                  }}</span>
                </div>
                <div class="resources__type-date">{{ rate.date }}</div>
              </div>
              <div class="resources__type-previous resources__type-column">
                <div class="resources__header resources__header--mobile">
                  Предыдущий месяц</div>
                <div class="resources__type-value"
                  v-html="rate.prevVal">
                </div>
              </div>
              <div class="resources__type-month resources__type-column">
                <div class="resources__header resources__header--mobile">Дата
                  последнего показания</div>
                <div class="resources__type-value">{{ rate.lastDate }}</div>
              </div>
            </div>
          </div>
          <div class="resources__type resources__type-heating"
            v-if="resource.name === 'Отопление'">
            <div class="resources__type-row"
              v-for="rate in resource.rates">
              <div class="resources__type-column-name resources__type-column">
                <div class="resources__type-dimension">
                  <div class="resources__type-square"
                    :style="{ background: rate.color }"></div>
                  <div class="resources__type-title">{{ resource.label }}
                    <svg @click="$emit('onEditCounter', resource)"
                      class="resources__type-edit-svg"
                      v-svg
                      size="10 10"
                      symbol="edit"></svg>
                  </div>
                </div>
                <div class="resources__type-info">
                  <div class="resources__counter">Счетчик №{{
                    resource.serialNumber
                  }}</div>
                  <div class="resources__virification-date">Поверка {{
                    resource.checkDate
                  }}</div>
                </div>
              </div>
              <div class="resources__type-current resources__type-column">
                <div class="resources__header resources__header--mobile">Текущие
                  показания, дата синхронизации</div>
                <div class="resources__type-value">
                  <span v-html="rate.curVal"></span>
                  <span class="resources__type-changes">{{
                    rate.changes
                  }}</span>
                </div>
                <div class="resources__type-date">{{ rate.date }}</div>
              </div>
              <div class="resources__type-previous resources__type-column">
                <div class="resources__header resources__header--mobile">
                  Предыдущий месяц</div>
                <div class="resources__type-value"
                  v-html="rate.prevVal">
                </div>
              </div>
              <div class="resources__type-month resources__type-column">
                <div class="resources__header resources__header--mobile">Дата
                  последнего показания</div>
                <div class="resources__type-value">{{ rate.lastDate }}</div>
              </div>
            </div>
          </div>
        </template>
      </div>
    </div>
    <resources-diagram :premise="premise"
      :types="premiseRef" />
  </div>
  <empty-info v-else>
    <p>
      По квартире <b>{{ premise.UF_NUMBER }}</b> пока нет показаний объектов учёта.
    </p>
  </empty-info>
</template>

<script>
import { ref, watch } from 'vue'
import dayjs from "dayjs";
import dateHelper from "../../../helpers/dateHelper";
import ResourcesDiagram from '../resources-diagram/resources-diagram.vue';
import EmptyInfo from "../../empty-info/empty-info";

export default {
  name: 'resources-premise',
  components: {EmptyInfo, ResourcesDiagram },
  emits: ['onEditCounter', 'loaded'],
  props: {
    premise: {
      type: Object,
      required: true,
      default: false,
    },
    counters: {
      type: Array,
      required: true,
      default: false,
    },
  },
  setup(props, { emit }) {
    const premiseRef = ref({})

    const getPremiseTitle = (premise) => {
      let roomPlaceholder = premise?.UF_ROOM_COUNT ? premise?.UF_ROOM_COUNT + 'к-квартира в ' : 'Квартира в '
      return roomPlaceholder + premise?.floor.approach.house.complex.UF_NAME
    }

    const getLocalAddress = (premise) => {
      return 'квартира №' + premise?.UF_NUMBER + ', этаж ' + premise?.floor?.UF_NUMBER + ', подъезд ' + premise?.floor?.approach?.UF_NUMBER
    }

    const getCityAddress = (premise) => {
      return premise?.floor?.approach?.house?.UF_NAME
    }

    const getColor = (rate) => {
      switch (rate) {
        case 'Дневной':
          return 'linear-gradient(180.31deg, #F7B26E 0%, #F79C6E 100%)'
        case 'Ночной':
          return 'linear-gradient(180.15deg, #6EA7F7 0%, #6E90F7 100%)'
        case 'Пиковый':
          return 'linear-gradient(180.15deg, #9A6EF7 0%, #896EF7 100%)'
        case 'Холодная':
          return 'linear-gradient(180.48deg, #6ECEF7 0%, #6EB7F7 100%)'
        case 'Горячая':
          return 'linear-gradient(180deg, #F7856E 0%, #F76E6E 100%)'
        case 'Основное':
          return 'linear-gradient(180.48deg, #ADD048 0%, #8CB31A 100%)'
      }
    }

    const defaultCounters = [
      {
        name: 'Электроэнергия',
        serialNumber: '—',
        checkDate: '—',
        label: 'Электроэнергия',
        rates: [
          {
            color: getColor('Дневной'),
            curVal: '—',
            changes: null,
            date: '—',
            lastDate: '—',
            name: 'Дневной',
            prevVal: '—'
          },
          {
            color: getColor('Ночной'),
            curVal: '—',
            changes: null,
            date: '—',
            lastDate: '—',
            name: 'Ночной',
            prevVal: '—'
          },
          {
            color: getColor('Пиковый'),
            curVal: '—',
            changes: null,
            date: '—',
            lastDate: '—',
            name: 'Пиковый',
            prevVal: '—'
          }
        ],
      },
      {
        name: 'Водоснабжение',
        serialNumber: '—',
        checkDate: '—',
        label: '—',
        rates: [
          {
            color: getColor("Холодная"),
            curVal: '—',
            checkDate: '—',
            changes: null,
            date: '—',
            lastDate: '—',
            name: 'Холодная',
            prevVal: '—',
            label: 'Холодная',
            serialNumber: '—',
          },
          {
            color: getColor('Горячая'),
            curVal: '—',
            changes: null,
            date: '—',
            lastDate: '—',
            name: 'Горячая',
            prevVal: '—',
            label: 'Горячая',
            serialNumber: '—',
          },
        ],
      },
      {
        name: 'Отопление',
        serialNumber: '—',
        checkDate: '—',
        label: 'Отопление',
        rates: [
          {
            color: getColor('Основное'),
            curVal: '—',
            changes: null,
            date: '—',
            lastDate: '—',
            name: 'Основное',
            prevVal: '—',
          }
        ],
      },
    ]

    const fillCounter = (counter, newCounter) => {

      newCounter.checkDate = dateHelper(counter.UF_VERIFICATION_DATE, 'DD MMMM YYYY')
      newCounter.serialNumber = counter.UF_SERIAL_NUMBER
      newCounter.label = counter.UF_CUSTOM_TITLE ? counter.UF_CUSTOM_TITLE : newCounter.name
      newCounter.id = counter.ID

      counter.meterReadings.data.map(rate => {
        let rateToFill = newCounter.rates.find(r => r.name === rate.rate.data.UF_TITLE)
        let readingsLastMonthfiltered = counter.meterReadingsLastMonth.data.filter(row => row.rate.data.UF_TITLE === rate.rate.data.UF_TITLE)
        let readingsLastMonth = readingsLastMonthfiltered.length ? readingsLastMonthfiltered.reduce((prev, current) => Date.parse(prev.UF_DATE) > Date.parse(current.UF_DATE) ? prev : current) : []

        rateToFill.value = getDecimalPresition(+rate.UF_VALUE)
        rateToFill.measurement = getMeasurement(counter.counter.data.type.data.UF_MEASUREMENT)
        rateToFill.changes = getChanges(rate.UF_DIFFERENCE_READINGS)
        rateToFill.date = getDate(rate.UF_DATE, 'DD MMMM в HH:mm')
        rateToFill.curVal = getValue(rateToFill.value, rateToFill.measurement)
        rateToFill.prevVal = readingsLastMonth ? getValue(getDecimalPresition(readingsLastMonth.UF_VALUE), rateToFill.measurement) : '—'
        rateToFill.lastDate = readingsLastMonth ? dateHelper(readingsLastMonth.UF_DATE, 'DD MMMM') : '—'

        if (newCounter.name === 'Водоснабжение') {
          rateToFill.id = counter.ID
          rateToFill.label = counter.UF_CUSTOM_TITLE ? counter.UF_CUSTOM_TITLE : rateToFill.name
          rateToFill.checkDate = dateHelper(counter.UF_VERIFICATION_DATE, 'DD MMMM YYYY')
          rateToFill.serialNumber = counter.UF_SERIAL_NUMBER
        }
      })
      return newCounter
    }

    const getDecimalPresition = (n) => {
      return typeof n === 'number' ? Math.round(n * 10) / 10 : n
    }

    const substractNumbers = (a, b) => {
      return (a * 10 - b * 10) / 10
    }

    const getMeasurement = (str) => {
      let i = str.indexOf(str.match(/\d+/))
      return i !== -1 ? { value: str.slice(0, i), index: str.slice(i) } : { value: str, index: '' }
    }

    const getDate = (date, format) => {
      return dayjs().format('DD MM YYYY') === dayjs(date).format('DD MM YYYY')
        ? 'сегодня в ' + dateHelper(date, 'HH:mm')
        : dateHelper(date, format)
    }

    const getValue = (value, measurement) => {
      if (value) {
        return `${value} ${measurement.value}<sup>${measurement.index}</sup>`
      }
      else {
        return '—'
      }
    }

    const getChanges = (changes) => {
      return changes ? ` (+${changes})` : ''
    }

    const sortRates = (info) => {
      info = info.map(type => {
        switch (type.name) {
          case 'Электроэнергия':
            type.rates.sort((a, b) => {
              return a.name < b.name ? -1 : a.name > b.name ? 1 : 0
            })
            break
          case 'Водоснабжение':
            type.rates.sort((a, b) => {
              return a.name < b.name ? 1 : a.name > b.name ? -1 : 0
            })
            break
        }
        return type
      })
      return info
    }

    const sortCounters = counters => {
      counters = counters.sort((a, b) => {
        if (a.name === 'Электроэнергия' && b.name === 'Водоснабжение' || a.name === 'Водоснабжение' && b.name === 'Отопление') {
          return -1
        }
        else return 1
      })
      return counters
    }

    watch(() => props.counters, (val) => {
      if (val.length) {
        let res
        res = JSON.parse(JSON.stringify(defaultCounters))
        res = val.map(counter => {
          let counterToFill = res.find(count => count.name === counter.counter.data.type.data.UF_TITLE)
          return fillCounter(counter, counterToFill)
        })
        res.splice(res.findLastIndex(counter => counter.name === 'Водоснабжение'), 1)
        res = sortCounters(res)
        res = sortRates(res)
        premiseRef.value = res
        emit('loaded')
      }
    }, { deep: true })

    return {
      premiseRef,
      getPremiseTitle,
      getLocalAddress,
      getCityAddress,
      getDecimalPresition,
      substractNumbers,
      getMeasurement,
      getDate,
    }
  }
}
</script>

<style lang="scss" src="./resources-premise.scss" scoped/>
