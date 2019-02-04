<template>
<v-touch v-on:press="longPress" class="device">

  <div class="icon on" :style="'-webkit-mask-image: url('+$homey._baseUrl+device.iconObj.url+')'"></div>
  <div class="name">{{device.name}}</div>

  <div class="info" v-if="device.capabilitiesObj.dim">{{device.capabilitiesObj.dim.value > 0 ? 'OPEN' : 'CLOSED'}} <span v-if="device.capabilitiesObj.dim.value"> - {{Number((device.capabilitiesObj.dim.value*100).toFixed(0))}}%</span></div>
  <div class="info" style="text-transform:uppercase;" v-else>{{device.capabilitiesObj.windowcoverings_state.value || 'Idle'}}</div>
  <div class="battery" v-if="device.capabilitiesObj.measure_battery && device.capabilitiesObj.measure_battery.value !== null">
      <q-icon color="teal" v-if="device.capabilitiesObj.measure_battery.value > 80" name="fa-battery-full" />
      <q-icon color="teal" v-else-if="device.capabilitiesObj.measure_battery.value < 81 && device.capabilitiesObj.measure_battery.value > 50" name="fa-battery-three-quarters" />
      <q-icon color="teal" v-else-if="device.capabilitiesObj.measure_battery.value < 51 && device.capabilitiesObj.measure_battery.value > 25" name="fa-battery-half" />
      <q-icon color="orange" v-else-if="device.capabilitiesObj.measure_battery.value < 56 && device.capabilitiesObj.measure_battery.value > 10" name="fa-battery-quarter" />
      <q-icon color="red" v-else-if="device.capabilitiesObj.measure_battery.value < 10" name="fa-battery-empty" />
  </div>

  <q-modal no-backdrop-dismiss style="background-color: rgba(0, 0, 0, 0.85)" class="device-modal" :content-css="{background: 'rgba(0, 0, 0, 0)', boxShadow: '0 0 0 0', border: '0 0 0 0'}" v-model="showModal" minimized>
    <q-list no-border>
      <q-list-header class="text-teal">{{device.name}}</q-list-header>
      <q-item v-if="device.capabilitiesObj.dim">
        <q-item-side class="text-white">
          {{device.capabilitiesObj.dim.title}}
        </q-item-side>
        <q-item-main class="text-right">
          <q-slider color="teal" v-if="device.capabilitiesObj.dim"
                                 :value="device.capabilitiesObj.dim.value"
                                 :min="device.capabilitiesObj.dim.min"
                                 :max="device.capabilitiesObj.dim.max"
                                 :step="device.capabilitiesObj.dim.step"
                                 @change="setDim" />
        </q-item-main>
      </q-item>

      <q-item v-if="device.capabilitiesObj.onoff">
        <q-item-side class="text-white">
          {{device.capabilitiesObj.onoff.title}}
        </q-item-side>
        <q-item-main class="text-right">
          <q-toggle icon="fa-power-off" color="teal" v-model="device.capabilitiesObj.onoff.value" @blur="setOnoff" @focus="setOnoff" />
        </q-item-main>
      </q-item>

      <q-item v-if="device.capabilitiesObj.windowcoverings_state">
        <q-item-side class="text-white">
          {{device.capabilitiesObj.windowcoverings_state.title}}
        </q-item-side>
        <q-item-main class="text-right" style="">

        </q-item-main>
      </q-item>
      <q-item v-if="device.capabilitiesObj.windowcoverings_state">
        <q-item-side class="text-white">
            <q-btn small @click="setState('up')" :color="device.capabilitiesObj.windowcoverings_state.value === 'up' ? 'teal' : 'neutral'"><q-icon name="fa-chevron-up" /></q-btn>
            <q-btn @click="setState('idle')" :color="device.capabilitiesObj.windowcoverings_state.value === 'idle' ? 'teal' : 'neutral'" small><q-icon name="fa-pause" /></q-btn>
            <q-btn @click="setState('down')" small :color="device.capabilitiesObj.windowcoverings_state.value === 'down' ? 'teal' : 'neutral'"><q-icon name="fa-chevron-down" /></q-btn>
        </q-item-side>
        <q-item-main class="text-right" style="">

        </q-item-main>
      </q-item>

      <q-item v-for="(value, key) in device.capabilitiesObj" :key="device.id + key" v-if="value.getable && value.units || key.substring(0, 5) == 'alarm'">
        <q-item-side class="text-white" v-if="!value.options.title">
          {{value.title}}
        </q-item-side>
        <q-item-side class="text-white" v-else>
          {{value.options.title}}
        </q-item-side>
        <q-item-main class="text-right text-white">
          <span v-if="typeof(device.capabilitiesObj[key].value) === 'boolean'"><q-icon v-if="!device.capabilitiesObj[key].value" color="green" name="fa-check" /><q-icon v-else color="red" name="fa-exclamation-triangle" /></span>
          <span v-else> <span v-if="device.capabilitiesObj[key].value != null">{{device.capabilitiesObj[key].value}}</span><span v-else>-</span> <span v-if="device.capabilitiesObj[key].units">{{device.capabilitiesObj[key].units}}</span></span>
        </q-item-main>
      </q-item>

    </q-list>
    <center>
      <q-btn style="margin-top:50px;" v-on:click="modalState(false)" icon="close" outline color="white">close</q-btn>
    </center>
  </q-modal>
</v-touch>
</template>

<script>
import _ from 'lodash'

export default {
  props: ['device'],
  data () {
    return {
      showModal: false
    }
  },
  methods: {
    longPress (ev) {
      if (navigator.vibrate) {
        navigator.vibrate(150)
      }
      this.showModal = true
    },
    setBlind () {
      if (this.device.capabilitiesObj.dim) {
        if (this.device.capabilitiesObj.dim.value > 0) {
          this.device.setCapabilityValue('dim', 0)
        } else {
          this.device.setCapabilityValue('dim', 1)
        }
      }
    },
    setState (value) {
      this.device.setCapabilityValue('windowcoverings_state', value)
    },
    setDim: _.debounce(function (value) {
      console.log(value)
      this.device.setCapabilityValue('dim', value)
    }, 100),
    modalState (state) {
      this.showModal = state
    }
  },
  mounted () {

  },
  computed: {
    blindState () {
      if (this.device.state.dim > 0) {
        return 'OPEN'
      } else {
        return 'CLOSED'
      }
    }
  }
}

</script>

  <style lang="stylus" scoped>
  @import '~variables'

  .device
    height 90px
    margin 5px
    text-align left
    background-color rgba(0, 0, 0, 0.5)
    border-radius 10px
    position relative
    .icon
      height 38px
      width 38px
      position relative
      top 7px
      left 7px
      -webkit-mask-size contain
      -webkit-mask-position top left
      -webkit-mask-repeat no-repeat
    .info
      position absolute
      left 7px
      bottom 7px
      font-size 0.8em
      color $secondary
      font-weight 300
      max-width 100%
      overflow hidden
      text-align left
      text-overflow ellipsis
      white-space nowrap
    .battery
      position absolute
      top 3px
      right 7px
    .off
      background-color $neutral
      opacity 0.2
    .on
      background-color $teal
    .name
      position absolute
      bottom 24px
      left 7px
      right 7px
      color $neutral
      font-size 0.8em
      max-width 100%
      overflow hidden
      text-align left
      text-overflow ellipsis
      white-space nowrap

  </style>
