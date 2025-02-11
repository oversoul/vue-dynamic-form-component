<template>
  <div class="dynamic-form" :style="style">
    <el-form
      v-if="_value"
      ref="dynamic-form"
      :model="_value"
      :rules="descriptors"
      enctype="multipart/form-data"
      >
      <dynamic-form-item
        v-for="(descriptor, key) in descriptors"
        v-model="_value[key]"
        :key="key"
        :lang="lang"
        :label="findTypeDescriptor(descriptor).label || key"
        :prop="key"
        :label-width="labelWidth"
        :descriptor="descriptor"
        :language="language"
        :size="size"
        :onDragEndCallback="onDragEnd"
        :background-color="backgroundColor"
        :bg-color-offset="bgColorOffset">
      </dynamic-form-item>
      <el-form-item v-if="$slots.operations" class="operations" :label-width="labelWidth">
        <slot name="operations"></slot>
      </el-form-item>
    </el-form>
  </div>
</template>

<script>
import DynamicFormItem from '../dynamic-form-item/form-item'
import { isComplexType, getLabelWidth, findTypeDescriptor } from '../utils'
import i18n from '../i18n'

export default {
  name: 'dynamic-form',
  props: {
    value: {
      type: Object,
      required: true
    },
    lang: {
      type: String,
      default: 'zh_CN'
    },
    /**
     * custom languages, format refer to packages/i18n.js
     */
    languages: Object,
    /**
     * descriptor of value, extend from https://github.com/yiminghe/async-validator
     */
    descriptors: {
      type: Object,
      required: true
    },
    /**
     * size of the input component
     */
    size: {
      type: String,
      default: 'small'
    },
    /**
     * background-color of form
     */
    backgroundColor: {
      type: String,
      default: '#FFFFFF'
    },
    /**
     * font-size of form
     */
    fontSize: {
      type: Number,
      default: 14
    },
    /**
     * darken sub-form's background-color with offset while get positive integer
     */
    bgColorOffset: {
      type: Number,
      default: 8
    },

    onDragEndCallback: {
      type: Function,
      default: () => {}
    }
  },
  components: {
    DynamicFormItem
  },
  computed: {
    _value: {
      get () {
        return this.value
      },
      set (value) {
        this.$emit('input', value)
      }
    },
    labelWidth () {
      return getLabelWidth(this.descriptors, this.fontSize)
    },
    style () {
      const style = {
        fontSize: `${this.fontSize}px`,
        backgroundColor: this.backgroundColor
      }
      return style
    },
    language () {
      return (this.languages || i18n)[this.lang]
    }
  },
  data () {
    return {
      i18n
    }
  },
  created () {
    this.init()
    // this.$bus.$on('onImageRemove', this.$listeners.onImageRemove)
    // this.$bus.$on('onImagePreview', this.$listeners.onImagePreview)
  },
  methods: {
    findTypeDescriptor,
    init () {
      this.initValue()
    },
    initValue () {
      for (const key in this.descriptors) {
        this.setValueKey(this, this._value, key, this.descriptors[key])
      }
    },
    setValueKey (target, value, key, descriptor) {
      if (isComplexType(descriptor.type)) {
        if (descriptor.type === 'object') {
          // object
          if (descriptor.fields) {
            // normal object
            if (value[key] === undefined) {
              target.$set(value, key, {})
            }
            for (const _key in descriptor.fields) {
              target.setValueKey(target, value[key], _key, descriptor.fields[_key])
            }
          } else {
            // hashmap
            if (value[key] === undefined) {
              target.$set(value, key, {})
            }
          }
        } else {
          // array
          if (value[key] === undefined) {
            target.$set(value, key, [])
          }
        }
      } else {
        if (value[key] === undefined) {
          target.$set(value, key, null)
        }
      }
    },
    validate (func) {
      if (typeof func === 'function') {
        this.$refs['dynamic-form'].validate(valid => {
          func(valid)
        })
      } else {
        return new Promise((resolve, reject) => {
          this.$refs['dynamic-form'].validate(valid => {
            return resolve(valid)
          })

          return reject()
        })
      }
    },
    resetFields () {
      this.$refs['dynamic-form'].resetFields()
    },
    clearValidate () {
      this.$refs['dynamic-form'].clearValidate()
    },
    onDragEnd(arr) {
      console.log('form');
    },
  }
}
</script>

<style lang="scss" scoped>
</style>
