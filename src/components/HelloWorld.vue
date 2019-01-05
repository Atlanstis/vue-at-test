<template>
  <div>
    <vue-at></vue-at>
    <el-input
      type="textarea"
      ref="textArea"
      v-model="textarea"
      placeholder="请输入"
      @input="mention"
      >
      </el-input>
      <pre>
        <code v-text="textarea"></code>
      </pre>
  </div>
</template>

<script>
import VueAt from './vue-at'
import getCaretCoordinates from 'textarea-caret'

export default {
  components: {VueAt},
  data () {
    return {
      textarea: ''
    }
  },
  methods: {
    mention () {
      let el = this.$el.querySelector('textarea')
      console.log(el.selectionEnd)
      const rect = getCaretCoordinates(el, el.selectionEnd)
      console.log(rect)
    }
  }
}
</script>

<style scoped>
</style>
