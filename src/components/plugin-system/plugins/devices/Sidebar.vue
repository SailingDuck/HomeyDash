<template>
<div>
  <q-inner-loading :visible="!loaded">
    <q-spinner size="50px" color="teal" />
  </q-inner-loading>
    <div v-if="loaded" v-for="zone in sortByIndex(nestedZones)" :key="zone.id">
      <q-item exact v-bind:key="zone.id" :to="{ name: 'Devices', params: { zone: zone.id } }">
        <q-item-side>
          <i class="material-icons" style="height:24px;padding:2px;display:inline-block;">{{ mapZones(zone.icon)}}</i>
        </q-item-side>
        <q-item-main :label="zone.name" />
      </q-item>
      <div v-for="child in sortByIndex(zone.children)" :key="child.id">
        <q-item exact v-bind:key="child.id" :to="{ name: 'Devices', params: { zone: child.id } }">
          <q-item-side style="padding-left:35px;">
            <i class="material-icons" style="height:24px;padding:2px;display:inline-block;">{{ mapZones(child.icon)}}</i>
          </q-item-side>
          <q-item-main :label="child.name" />
        </q-item>
        <div v-for="child in sortByIndex(child.children)" :key="child.id">
          <q-item exact v-bind:key="child.id" :to="{ name: 'Devices', params: { zone: child.id } }">
            <q-item-side style="padding-left:70px;">
              <i class="material-icons" style="height:24px;padding:2px;display:inline-block;">{{ mapZones(child.icon)}}</i>
            </q-item-side>
            <q-item-main :label="child.name" />
          </q-item>
        </div>
      </div>
    </div>
</div>
</template>

<script>
import _ from 'lodash'

export default {
  data () {
    return {
      zones: {},
      nestedZones: {},
      loaded: false
    }
  },
  async created () {
    if (this.nestedZones) {
      setTimeout(() => {
        this.loaded = true
      }, 500)
    }

    this.zones = await this.$homey.zones.getZones({
      icons: 'png'
    })
    this.nestedZones = this.buildTree(this.zones, 'id', 'parent', 'children')
  },
  methods: {
    buildTree (flatList, idFieldName, parentKeyFieldName, fieldNameForChildren) {
      var rootElements = []
      var lookup = {}

      _.forEach(flatList, function (flatItem) {
        var itemId = flatItem[idFieldName]
        lookup[itemId] = flatItem
        flatItem[fieldNameForChildren] = []
      })

      _.forEach(flatList, function (flatItem) {
        var parentKey = flatItem[parentKeyFieldName]
        if (parentKey != null) {
          var parentObject = lookup[flatItem[parentKeyFieldName]]
          if (parentObject) {
            parentObject[fieldNameForChildren].push(flatItem)
          } else {
            rootElements.push(flatItem)
          }
        } else {
          rootElements.push(flatItem)
        }
      })
      let sortedView = _.sortBy(rootElements, [function (o) {
        return o.index
      }])
      return sortedView
    },
    sortByIndex (list) {
      return _.orderBy(list, 'name', 'asc')
    },
    mapZones (icon) {
      switch (icon) {
        case 'default':
          return 'store'
        case 'bed':
          return 'hotel'
        case 'books':
          return 'book'
        case 'garden':
          return 'local_florist'
        case 'home':
          return 'home'
        case 'kitchen':
          return 'restaurant'
        case 'living':
          return 'weekend'
        case 'roof':
          return 'eject'
        case 'shower':
          return 'grain'
        case 'stairs-down':
          return 'arrow_downward'
        case 'stairs-up':
          return 'arrow_upward'
        case 'toilet':
          return 'wc'
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="stylus">
@import '~variables'

a
  color $primary
  :hover
    background-color $tertiary

</style>
