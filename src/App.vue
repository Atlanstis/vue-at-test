<template>
  <div id="app">
    <at-ta :members="members" name-key="name" v-model="text" @at="at" ref="at" :isFromNet="true" :keyAlias="'key'" :nameAlias="'name'">
      <!-- custom: with avatars -->
      <template slot="item" slot-scope="s">
        <img :src="s.item.avatar">
        <span v-text="s.item.name"></span>
      </template>
      <el-input
        type="textarea"
        :rows="2"
        placeholder="请输入内容"
        v-model="text">
      </el-input>
    </at-ta>
    {{text}}
    <button @click="test">test</button>
  </div>
</template>

<script>
import AtTa from './AtTextarea.vue'

let members = [
  /* eslint-disable */
  "Roxie Miles","grace.carroll",
  "小浩",
  "Helena Perez","melvin.miller",
  "椿木",
  "myrtie.green","elsie.graham","Elva Neal",
  "肖逵",
  "amy.sandoval","katie.leonard","lottie.hamilton",
  /* eslint-enable */
]
members = members.map((v, i) => {
  return {
    avatar: `https://randomuser.me/api/portraits/men/${i % 5}.jpg`,
    name: v,
    key: i
  }
})

export default {
  components: { AtTa },
  name: 'app',
  data () {
    return {
      members: [],
      text: ''
    }
  },
  methods: {
    at (val) {
      console.log(val)
      setTimeout(() => {
        if (val === '') {
          this.$refs.at.updateMembers(members)
        } else {
          let list = members.filter((v) => {
            return v.name.indexOf(val) > -1
          })
          this.$refs.at.updateMembers(list)
        }
      }, 20)
    },
    test () {
      this.$refs.at.test()
    }
  }
}
</script>

<style>
html {
  width: 100%;
  height: 100%;
}
body {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0;
}

/* override styles */
#app .atwho-li {
  padding: 0 4px;
}
#app .atwho-li img {
  height: 100%;
  width: auto;
  transform: scale(.8);
}
#app .atwho-li span {
  padding-left: 8px;
}
</style>
