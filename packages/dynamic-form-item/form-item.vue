<template>
  <el-form-item
    v-show="!descriptor.hidden"
    class="dynamic-form-item"
    :ref="prop"
    :label="labelWidth === '0px' ? '' : (label || prop)"
    :prop="prop"
    :size="size"
    :language="language"
    :rules="descriptor"
    :required="typeDescriptor.required"
    :label-width="labelWidth">
    <dynamic-input
      v-if="!isComplexType(typeDescriptor.type)"
      v-model="_value"
      :size="size"
      :prop="prop ? prop : key"
      :background-color="subFormBackgroundColor"
      :language="language"
      :descriptor="typeDescriptor">
    </dynamic-input>
    <!-- complex type, object or array -->
    <template v-else>
      <!-- normal object or hashmap object -->
      <template v-if="typeDescriptor.type === 'object'">
        <!-- normal object with known keys -->
        <div
          v-if="!typeDescriptor.defaultField"
          class="sub-dynamic-form"
          :style="{backgroundColor: subFormBackgroundColor}">
          <dynamic-form-item
            v-for="(_descriptor, key) in typeDescriptor.fields"
            v-model="_value[key]"
            :key="key"
            :label="(findTypeDescriptor(_descriptor)).label || key"
            :prop="prop ? prop + '.' + key : key"
            :descriptor="_descriptor"
            :language="language"
            :label-width="getLabelWidth(typeDescriptor.fields, fontSize)"
            :background-color="subFormBackgroundColor">
          </dynamic-form-item>
        </div>
        <!-- hashmap object -->
        <div
          v-else
          class="sub-dynamic-form hashmap"
          :style="{backgroundColor: subFormBackgroundColor}">
          <dynamic-form-item
            v-for="(temp, key) in _value"
            v-model="_value[key]"
            :ref="prop + '.' + key"
            :key="key"
            :label="key"
            :prop="prop ? prop + '.' + key : key"
            :deletable="true"
            :descriptor="typeDescriptor.defaultField"
            :language="language"
            :label-width="getLabelWidth(_value, fontSize)"
            :background-color="subFormBackgroundColor"
            @delete="deleteKey(key)">
          </dynamic-form-item>
          <el-form-item>
            <div class="add-key-input-group">
              <el-input v-model="hashMapKey" :placeholder="language.addKeyPlaceholder" :size="size"></el-input>
              <el-button type="primary" icon="el-icon-plus" :size="size" :disabled="!hashMapKey || _value[hashMapKey] !== undefined" @click="addHashMapKey" plain>{{ language.addKeyButtonText }}</el-button>
            </div>
          </el-form-item>
        </div>
      </template>
      <!-- array -->
      <template v-else>
        <div v-if="typeDescriptor.defaultField.type === 'enum' && typeDescriptor.defaultField.multiple" class="multi-select">
          <dynamic-input
            v-model="_value"
            :size="size"
            :descriptor="typeDescriptor.defaultField">
          </dynamic-input>
        </div>
        <div v-else class="sub-dynamic-form array" :style="{backgroundColor: subFormBackgroundColor}">
          <draggable v-model="_value" @end="onDragEnd(_value, typeDescriptor.onDragEnd)" handle=".drag-button">
          <dynamic-form-item
            v-for="(temp, key) in _value"
            v-model="_value[key]"
            :key="key"
            :prop="prop ? prop + '.' + key : key"
            :deletable="true"
            :draggable="true"
            :descriptor="typeDescriptor.defaultField"
            :language="language"
            label-width="0px"
            :background-color="subFormBackgroundColor"
            @delete="deleteItem(key)">
          </dynamic-form-item>
          </draggable>
          <div class="add-key-input-group">
            <el-button type="primary" icon="el-icon-plus" :size="size" @click="addArrayItem" plain>{{ language.addArrayItemButtonText }}</el-button>
          </div>
        </div>
      </template>
    </template>
    <el-button v-if="deletable" class="delete-button" type="text" icon="el-icon-close" @click="emitDelete"></el-button>
    <i v-if="draggable" class="el-icon-rank drag-button"></i>
  </el-form-item>
</template>

<script>
import { isComplexType, getLabelWidth, darkenColor, parseDescriptor, findTypeDescriptor } from '../utils'
import DynamicInput from '../dynamic-input/input'
import draggable from 'vuedraggable'

export default {
  name: 'dynamic-form-item',
  props: {
    value: {
      required: true
    },
    prop: {
      type: String,
      default: ''
    },
    label: String,
    /**
     * descriptor of value, extend from https://github.com/yiminghe/async-validator
     */
    descriptor: {
      type: [Object, Array],
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
     * font-size of form
     */
    fontSize: {
      type: Number,
      default: 14
    },
    /**
     * background-color of form
     */
    backgroundColor: {
      type: String,
      default: '#FFFFFF'
    },
    /**
     * darken sub-form's background-color with offset if got positive integer
     */
    bgColorOffset: {
      type: Number,
      default: 8
    },
    deletable: {
      type: Boolean,
      default: false
    },
    draggable: {
      type: Boolean,
      default: false
    },
    onDragEndCallback: {
      type: Function,
      default: () => {}
    },
    labelWidth: String,
    language: Object
  },
  components: {
    DynamicInput,
    draggable,
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
    typeDescriptor () {
      return findTypeDescriptor(this.descriptor)
    },
    subFormBackgroundColor () {
      return this.bgColorOffset ? darkenColor(this.backgroundColor, this.bgColorOffset) : 'none'
    }
  },
  data () {
    return {
      hashMapKey: ''
    }
  },
  created () {
  },
  methods: {
    onDragEnd(value, callback) {
      if ( typeof callback !== 'function' ) return;
      callback(value)
    },
    isComplexType,
    getLabelWidth,
    findTypeDescriptor,
    clearValidate () {
      this.$refs[this.prop].clearValidate()
    },
    resetField () {
      this.$refs[this.prop].resetField()
    },
    addHashMapKey () {
      this.$set(this._value, this.hashMapKey, parseDescriptor(this.typeDescriptor.defaultField))
      this.hashMapKey = ''
      this.$refs[this.prop].resetField() // reset field to clear validate status while adding fist hashmap key
    },
    addArrayItem () {
      this._value.push(parseDescriptor(this.typeDescriptor.defaultField))
    },
    emitDelete () {
      this.$emit('delete')
    },
    deleteKey (key) {
      this.$delete(this._value, key)
    },
    deleteItem (index) {
      this._value.splice(index, 1)
    }
  }
}
</script>

<style lang="scss" scoped>
.sub-dynamic-form {
  border-radius: 5px;
  padding: 10px;
  .el-form-item:last-child {
    margin-bottom: 0;
  }
}
.add-key-input-group {
  display: flex;
  margin-top: 10px;

  .el-input {
    width: 250px;
    margin-right: 10px;
  }
}
.add-key-input-group:first-child {
  margin-top: 0;
}
.delete-button {
  position: absolute;
  top: 0;
  right: 5px;
  font-size: 20px;
  color: #F56C6C;
  padding: 5px 0;
}
.delete-button:hover {
  color: red;
}
.dynamic-input + .delete-button {
  top: auto;
  right: auto;
}
.drag-button {
  position: absolute;
  top: 40px;
  right: 5px;
  font-size: 20px;
  color: #606266;
  padding: 5px 0;
}
.drag-button:hover {
  color: #5cb6ff;
}
.dynamic-input + .drag-button {
  top: auto;
  right: auto;
}
</style>
