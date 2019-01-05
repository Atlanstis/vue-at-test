<template>
   <div>
    <at :members="members" v-model="html">
      <div contenteditable class="editor"></div>
    </at>
    <pre class="output">
      <code v-text="output1"></code>
    </pre>

    <at-ta :members="members" v-model="text" :filterMatch="filterMatch" :deleteMatch="deleteMatch">
      <textarea class="editor"></textarea>
    </at-ta>
    <pre class="output">
      <code v-text="output2"></code>
    </pre>
   </div>
</template>

<script>
import AtTa from 'vue-at/dist/vue-at-textarea'
import At from 'vue-at/dist/vue-at'

export default {
  components: { At, AtTa },
  data () {
    return {
      html: 'editable v-model: @Roxie Miles .<br>@grace.carroll @小浩 lol',
      text: '',
      members: ['Roxie Miles', 'grace.carroll', '小浩', '1111', '1111', '222', '333', '444']
    }
  },
  computed: {
    output1 () {
      return JSON.stringify({ html: this.html }, null, 2)
    },
    output2 () {
      return JSON.stringify({ text: this.text }, null, 2)
    }
  },
  methods: {
    filterMatch (name, chunk, at) {
      return name.toLowerCase().indexOf(chunk.toLowerCase()) > -1
    },
    deleteMatch (name, chunk) {
      return name === chunk
    }
  }
}
</script>

<style scoped>
  .editor {
    width: 300px;
    height: 40px;
  }
</style>
