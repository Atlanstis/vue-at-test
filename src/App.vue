<template>
  <div id="app">
    <at-ta :members="members" @at="at" ref="at" :keyAlias="'key'" :nameAlias="'name'" :deptAlias="'dept'" :affix="affix">
      <el-input
        type="textarea"
        :rows="2"
        placeholder="请输入内容">
      </el-input>
    </at-ta>
    <button @click="test">test</button>
    <button @click="clear">clear</button>
  </div>
</template>

<script>
import AtTa from './AtTextarea.vue'

let members = [
  /* eslint-disable */
  "Roxie Miles","Roxie Miles",
  "grace.carroll","grace.carroll",
  "小浩",
  "Helena Perez","melvin.miller",
  "椿木",
  "myrtie.green","elsie.graham","Elva Neal",
  "肖逵",
  "amy.sandoval","katie.leonard","lottie.hamilton",
  /* eslint-enable */
]
members = members.map((v, i) => {
  let dept = '研发部'
  if (i === 0) {
    dept = '财务部'
  }
  if (i === 2) {
    dept = '财务部'
  }
  return {
    name: v,
    key: i,
    dept: dept
  }
})

export default {
  components: { AtTa },
  name: 'app',
  data () {
    return {
      members: [],
      text: '',
      affix: {
        front: '<span>',
        end: '</span>'
      }
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
      let obj = this.$refs.at.getInfo()
      console.log(obj)
    },
    clear () {
      this.$refs.at.clear()
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

.el-textarea {
  width: 500px;
}
</style>
