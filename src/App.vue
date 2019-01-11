<template>
  <div id="app" class="template__at-who">
    <div>
      <at-who @at="at" ref="at" :keyAlias="'key'" :nameAlias="'name'" :deptAlias="'dept'" :affix="affix">
        <el-input
          type="textarea"
          :rows="2"
          placeholder="请输入内容">
        </el-input>
      </at-who>
      <div class="template__btns">
        <el-button @click="test">测试</el-button>
        <el-button @click="clear">清除</el-button>
      </div>
    </div>
  </div>
</template>

<script>
import AtWho from './AtTextarea'

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
  components: { AtWho },
  name: 'app',
  data () {
    return {
      members: [],
      text: '',
      affix: {
        front: '<span class="template_person">',
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
      this.$alert(obj.template, 'HTML', {
        dangerouslyUseHTMLString: true
      })
    },
    clear () {
      this.$refs.at.clear()
    }
  }
}
</script>

<style>
.template__at-who {
  width: 100%;
  height: 100%;
  margin-top: 200px;
  display: flex;
  align-items: center;
  justify-content: center;
}
.template__btns {
  text-align: center;
  margin-top: 20px;
}
.template_person {
  color: #FF8E3D;
}

.el-textarea {
  width: 500px;
}
</style>
