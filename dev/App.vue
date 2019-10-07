<template>
  <div id="app">
    <dynamic-form
      v-model="data"
      ref="dynamic-form"
      lang="en_US"
      @onImageRemove="test"
      :descriptors="descriptors">
      <template slot="operations">
        <el-button @click="reset">Reset</el-button>
        <el-button type="primary" @click="validate" plain>Validate</el-button>
      </template>
    </dynamic-form>
  </div>
</template>

<script>
import { Button } from 'element-ui'
import DynamicForm from '../packages/dynamic-form/index'
// import descriptors from './descriptor'

export default {
  name: 'app',
  components: {
    DynamicForm,
    ElButton: Button
  },
  data () {
    return {
      data: { },
      descriptors: {
        string: { type: 'string', required: true, disabled: true, placeholder: 'test', props: { autocomplete: 'on', type: 'textarea', rows: 4 } },
        url: { type: 'url', message: 'The url must be an url' },
        media_queue: {
          type: 'array',
          defaultField: {
            type: 'upload'
          }
        },
        select: {
          type: 'enum',
          filterable: true,
          options: [
            {
              value: 'Option1',
              label: 'Option1'
            }, {
              value: 'Option2',
              label: 'Option2'
            }, {
              value: 'Option3',
              label: 'Option3'
            }, {
              value: 'Option4',
              label: 'Option4'
            }, {
              value: 'Option5',
              label: 'Option5'
            }
          ],
        },

        object: {
          type: 'object',
          label: 'object label',
          fields: {
            boolean: { type: 'boolean', required: true },
            string: { type: 'string', required: true, hidden: false },
            date: { type: 'date', required: true },
            url: { type: 'url', message: 'The url must be an url', props: { placeholder: 'please input the url' } }
          }
        },
        hashmap: {
          type: 'object',
          label: 'hashmap label',
          defaultField: {
            type: 'string',
            required: true
          }
        },
        array: {
          type: 'array',
          label: 'array label',
          defaultField: {
            type: 'object',
            fields: {
              string: { type: 'string', required: true },
              url: { type: 'url', message: 'The url must be an url', props: { placeholder: 'please input the url' } },
              order: { type: 'number' },
              nextarr: {
                type: 'array',
                onDragEnd: this.done,
                draggable: true,
                defaultField: {
                  type: 'object',
                  fields: {
                    num: { type: 'number' },
                    order: { type: 'number' }
                  }
                }
              }
            }
          }
        },
        multiSelect: {
          type: 'array',
          label: 'array label',
          defaultField: {
            type: 'enum',
            multiple: true,
            enum: [0, 1, 2, 3],
            options: [
              { label: 'option-0', value: 0, disabled: true },
              { label: 'option-1', value: 1 },
              { label: 'option-2', value: 2 },
              { label: 'option-3', value: 3 }
            ]
          }
        }
      }
    }
  },
  watch: {
    'data.string' (value) {
      if (value) {
        this.descriptors.object.fields.string.hidden = true
      } else {
        this.descriptors.object.fields.string.hidden = false
      }
    }
  },
  methods: {
    reset () {
      this.$refs['dynamic-form'].resetFields()
    },
    validate () {
      console.log(JSON.parse(JSON.stringify(this.data)))
    },

    test(file, list) {
      console.log(file)
    },

    done(arr) {
      arr.map((el, index) => {
        arr[index].order = index + 1
      })
    },
    done2(val) {
      console.log('child')
    }

  },
  created () {}
}
</script>

<style lang="scss">
html, body {
  margin: 0;
  padding: 0;
}
#app {
  padding: 20px;
}
</style>
