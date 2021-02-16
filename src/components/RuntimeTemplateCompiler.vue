<template>
  <component :is="compiled" v-if="compiled" ref="child" />
</template>

<script>
'use strict'

import { compileToFunctions } from 'vue-template-compiler'
import { mergeAll, merge } from '../utils/helpers'
import config from '../utils/config'

export default {
  name: 'RuntimeTemplateCompiler',

  inheritAttrs: false,

  props: {
    parent: {
      type: Object,
      default: undefined
    },

    template: {
      type: String,
      default: config.defaultRuntimeTemplate
    },

    compilerOptions: {
      type: Object,
      default: () => config.defaultCompilerOptions
    },

    templateProps: {
      type: Object,
      default: () => ({})
    }
  },

  data() {
    return {
      isCompiling: false,
      compiled: null
    }
  },

  computed: {
    componentProps() {
      const data = [
        this.parentData,
        this.parentProps,
        this.parentComponent._provided
      ]
      return {
        filters: this.parentFilters,
        components: this.parentComponents,
        computed: this.parentComputed,
        methods: this.parentMethods,
        data: () => mergeAll(data)
      }
    },

    parentComponent() {
      return this.parent || this.$parent
    },

    parentData() {
      return this.parentComponent.$data || {}
    },

    parentProps() {
      return this.parentComponent.$props || {}
    },

    parentOptions() {
      return this.parentComponent.$options || {}
    },

    parentComputed() {
      return this.parentOptions.computed || {}
    },

    parentComponents() {
      return this.parentOptions.components || {}
    },

    parentMethods() {
      return this.parentOptions.methods || {}
    },

    parentFilters() {
      return this.parentOptions.filters || {}
    }
  },

  beforeMount() {
    this.compile()
  },
  mounted() {
    Object.keys(this.parentComponent.$data).forEach((key) => {
      try {
        if (this.parentComponent.$data[key] !== this.parentComponent)
          // avoid "Too much recursion" error, when using parent
          this.$watch(
            '$refs.child.' + key,
            (new_value) => (this.parentComponent.$data[key] = new_value),
            { deep: true }
          )
        // catch other cases which may not work this way (usually because of deep-check)
      } catch {
        console.warn('Cannot watch: ' + '$refs.child.' + key)
      }
    })
  },

  methods: {
    compile() {
      const component = compileToFunctions(this.template, this.compilerOptions)
      this.compiled = merge(component, this.componentProps, true)
    }
  }
}
</script>
