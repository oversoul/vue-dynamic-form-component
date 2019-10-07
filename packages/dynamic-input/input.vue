<template>
  <component
    class="dynamic-input"
    v-model="_value"
    v-if="!isSpecialType"
    v-bind="_bind"
    :is="name"
    :size="size">
  </component>
  <!-- integer, number, float type use el-input with v-model.number -->
  <el-input
    v-else-if="['integer', 'number', 'float'].includes(descriptor.type)"
    class="dynamic-input"
    v-model.number="_value"
    v-bind="_bind"
    :size="size">
  </el-input>
  <!-- enum type use el-select -->
  <el-select
    v-else-if="descriptor.type === 'enum'"
    class="dynamic-input"
    v-model="_value"
    v-bind="_bind"
    :class="{'multi-select': descriptor.multiple}"
    :size="size"
    :filterable="descriptor.filterable"
    :multiple="descriptor.multiple">
    <el-option v-for="option in _options" :key="option.label" :value="option.value" :label="option.label" :disabled="option.disabled"></el-option>
  </el-select>
  <!-- date type use el-date-picker -->
  <el-date-picker
    v-else-if="descriptor.type === 'date'"
    class="dynamic-input"
    type="datetime"
    v-model="_value"
    v-bind="_bind"
    :size="size">
  </el-date-picker>
  <div v-else-if="descriptor.type === 'upload'" class="file-upload">
    <div v-if="!_value" >
      <h2>Select an image</h2>
      <input type="file" @change="onFileChange">
    </div>
    <div v-else>
      <el-image v-bind="_bind" style="width: 100px; height: 100px" :src="_value"></el-image>
    </div>
  </div>
</template>

<script>
const TYPE_COMPONENT_MAP = {
  string: 'el-input',
  number: 'el-input-number',
  boolean: 'el-switch',
  regexp: 'el-input',
  integer: 'el-input-number',
  float: 'el-input-number',
  enum: 'el-select',
  url: 'el-input',
  upload: 'el-upload',
}

export default {
  name: 'dynamic-input',
  componentName: 'dynamic-input',
  props: {
    value: {
      required: true
    },
    size: {
      type: String,
      default: 'small'
    },
    descriptor: {
      type: Object,
      required: true
    },
    prop: {
      type: String
    }
  },
  components: {},
  computed: {
    _value: {
      get () {
        return this.value
      },
      set (value) {
        this.$emit('input', value)
      }
    },
    _options () {
      if (this.descriptor && this.descriptor.options instanceof Array) {
        return this.descriptor.options.map(item => {
          if (typeof item === 'string') {
            return { label: item, value: item }
          } else {
            return item
          }
        })
      } else {
        return []
      }
    },
    _bind () {
      let data = {};
      /**
       * Compatible with the version <= 2.2.0
       * These props is the first level prop of descriptor in old version
       */
      ['disabled', 'placeholder', 'autocomplete'].forEach(key => {
        if (typeof this.descriptor[key] !== 'undefined') {
          data[key] = this.descriptor[key]
        }
      })
      return Object.assign(data, this.descriptor.props)
    },
    isSpecialType () {
      return ['integer', 'float', 'number', 'enum', 'date', 'upload'].includes(this.descriptor.type)
    }
  },
  data () {
    return {
      name: '',
      dialogImageUrl: '',
      dialogVisible: false
    }
  },
  created () {
    this.init()
  },
  methods: {
    init () {
      this.name = TYPE_COMPONENT_MAP[this.descriptor.type] || 'el-input'
    },

    onFileChange(e) {
      var files = e.target.files || e.dataTransfer.files;
      if (!files.length)
        return;
      this.createImage(files[0]);
    },
    createImage(file) {
      var image = new Image();
      var reader = new FileReader();

      reader.onload = (e) => {
        this._value = e.target.result;
      };
      reader.readAsDataURL(file);
      if ( typeof this.descriptor.onFileChange === 'function' ) {
        this.descriptor.onFileChange(file, this.prop);
      }
    },
    removeImage: function (e) {
      this._value = '';
    },

    onImagePreview(file) {
      // this.$bus.$emit('onImagePreview', file)
    },
    onImageRemove(file, fileList) {
      // this.$bus.$emit('onImageRemove', { file, fileList })
    },

    handleImageExceed(files, fileList) {
      // this.$bus.$emit('onImagesExceed', { files, fileList })
    },
    handleRemove(file, fileList) {
      console.log(file, fileList);
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url;
      this.dialogVisible = true;
    },
    onFLChange(file, fileList) {
      this._value = fileList
    }
  }
}
</script>

<style lang="scss" scoped>
.dynamic-input {
  width: calc(100% - 60px);
}
.multi-select {
  width: calc(100% - 60px);
}
.file-upload {
  padding: 10px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  border-radius: 5px;
  min-height: 100px;
  text-align: center;
}
.file-upload h2 {
  margin: 0;
  font-size: 15px;
}
.file-upload .el-image {
  display: block;
  border-radius: 5px;
}
</style>
