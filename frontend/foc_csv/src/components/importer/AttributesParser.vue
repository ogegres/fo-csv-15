<template>
<div>
  <div class="form-group">
    <label class="label label-default">{{ $t('Attributes field') }}</label>
    <select class="form-control" v-model="attributesCSVField">
      <option :value="undefined">{{ $t('Not selected') }}</option>
      <option v-for="(csvField, idx) in csvFields" :key="idx" :value="idx">
        {{ csvField }}
      </option>
    </select>
  </div>

  <div class="form-group">
    <div class="checkbox">
      <label>
        <input type="checkbox" v-model="replaceAttributes"> {{ $t('Replace existing attributes') }}
      </label>
    </div>
  </div>

  <div class="form-group">
    <label class="label label-default">{{ $t('Attributes parser') }}</label>
    <select class="form-control" v-model="currentAttributeParserName">
      <option :value="null">{{ $t('Not selected') }}</option>
      <option v-for="(parser, key, idx) in attributeParsers" :key="idx" :value="key">
        {{ parser.title }}
      </option>
    </select>
  </div>

  <div class="form-group" v-if="showAttributeParserSettings" v-for="(option, key) in attributeParserOptions" :key="key">
    <label class="label label-default">{{ option.title }}</label>
    <component
      :is="getAttributeParserControl(option)"
      :value="attributeParserOptionData[key]"
      @change="setAttributeParserData([key, $event])"
    ></component>
  </div>

  <div class="form-group">
    <label class="label label-default">{{ $t('Default attributes group') }}</label>
    <autocomplete
      :url="attributesGroupUrl"
      requestType="get"
      input-class="form-control"
      v-model="defaultAttributesGroup"
    >
    </autocomplete>
  </div>
</div>
</template>

<script>
import { mapVuexModels } from 'vuex-models'
import { createNamespacedHelpers } from 'vuex'
import Autocomplete from 'autocomplete-vue'
import { ATTRIBUTES_GROUP_AUTOCOMPLETE_URL } from '@/api/routes'

import AttributeWidgets from './attributeWidgets'

const { mapActions, mapGetters } = createNamespacedHelpers('importer')

export default {
  components: {
    Autocomplete,
    ...AttributeWidgets
  },
  computed: {
    ...mapVuexModels([
      'attributeParser',
      'defaultAttributesGroup',
      'attributesCSVField',
      'replaceAttributes'
    ], 'importer'),
    ...mapGetters([
      'csvFields',
      'attributeParsers',
      'attributeParserOptionData',
      'currentAttributeParser',
      'attributeParserOptions'
    ]),
    attributesGroupUrl () {
      return this.$api.mkUrl(ATTRIBUTES_GROUP_AUTOCOMPLETE_URL)
    },
    currentAttributeParserName: {
      get () {
        return this.currentAttributeParser
      },
      set (newValue) {
        this.setAttributeParser(newValue)
      }
    },
    showAttributeParserSettings () {
      let settings = this.attributeParserOptions

      if (!settings || Object.keys(settings).length === 0) {
        return false
      }

      return true
    }
  },
  methods: {
    getAttributeParserControl (attribute) {
      if (attribute.type) {
        return `${attribute.type}-attribute`
      }
      return 'text-attribute'
    },
    ...mapActions([
      'setAttributeParserData',
      'setAttributeParser'
    ])
  }
}
</script>
