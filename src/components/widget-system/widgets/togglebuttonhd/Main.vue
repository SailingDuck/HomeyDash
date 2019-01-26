<template>
  <v-touch v-on:tap="switchDevice">
    <div class="onoff" v-if="!loading">
      <div class="icon" v-bind:style="{ maskImage: 'url(' + widget.settings.icon + ')', backgroundColor: widget.settings.offcolor }"></div>
      <h5>{{ widget.settings.name }}</h5>
      <small class="text-grey" v-if="widget.settings.room">{{ zone.name }}</small>
    </div>
  </v-touch>
</template>

<script>
export default {
  props: ['widget'],
  data () {
    return {
      device: {},
      zone: {},
      loading: true,
      state: false
    }
  },
  async mounted () {
    await this.getOnOffDevice()
    this.loading = false

    setTimeout(() => {
      this.updateBackgroundColor(this.state)
    }, 1000)
  },
  methods: {
    async getOnOffDevice () {
      this.device = await this.$homey.devices.getDevice({ id: this.widget.settings.onoff })
      this.zone = await this.$homey.zones.getZone({ id: this.device.zone })

      if (this.device.capabilitiesObj.onoff) {
        this.state = this.device.capabilitiesObj.onoff.value
      }
      this.device.on('$state', state => {
        this.state = this.device.capabilitiesObj.onoff.value
        this.updateBackgroundColor(this.state)
      })
    },
    async switchDevice () {
      if (this.device.capabilitiesObj.onoff) {
        this.device.setCapabilityValue('onoff', !this.device.capabilitiesObj.onoff.value)
        setTimeout(() => {
          this.updateBackgroundColor(this.device.capabilitiesObj.onoff.value)
        }, 1000)
      } else if (this.device.capabilitiesObj.button) {
        this.device.setCapabilityValue('button', true)
        this.$el.querySelector('.icon').style.backgroundColor = this.widget.settings.oncolor
        setTimeout(() => {
          this.$el.querySelector('.icon').style.backgroundColor = this.widget.settings.offcolor
        }, 2000)
      }
    },
    updateBackgroundColor (state) {
      if (state) {
        this.$el.querySelector('.icon').style.backgroundColor = this.widget.settings.oncolor
      } else {
        this.$el.querySelector('.icon').style.backgroundColor = this.widget.settings.offcolor
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
  @import '~variables'

  .onoff
    text-align center
    padding 6px

  .icon
    display block
    width auto
    height auto
    min-height 75px
    -webkit-mask-position center center
    mask-position center center
    -webkit-mask-repeat no-repeat
    mask-repeat no-repeat
    -webkit-mask-size auto 60px
    mask-size auto 60px
</style>
