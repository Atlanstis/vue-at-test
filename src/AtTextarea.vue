<script>
import {
  closest, scrollIntoView, getAtAndIndex
} from './util'
import AtTemplate from './AtTemplate.vue'
import At from './At.vue'
import getCaretCoordinates from 'textarea-caret'

export default {
  extends: At,
  name: 'AtTextarea',
  mixins: [AtTemplate],
  props: {
    value: {
      type: String, // value not required
      default: null
    },
    at: {
      type: String,
      default: null
    },
    ats: {
      type: Array,
      default: () => ['@']
    },
    suffix: {
      type: String,
      default: ' '
    },
    loop: {
      type: Boolean,
      default: true
    },
    allowSpaces: {
      type: Boolean,
      default: true
    },
    avoidEmail: {
      type: Boolean,
      default: true
    },
    hoverSelect: {
      type: Boolean,
      default: true
    },
    members: {
      type: Array,
      default: () => []
    },
    nameKey: {
      type: String,
      default: ''
    },
    filterMatch: {
      type: Function,
      default: (name, chunk, at) => {
        // match at lower-case
        return name.toLowerCase()
          .indexOf(chunk.toLowerCase()) > -1
      }
    },
    deleteMatch: {
      type: Function,
      default: (name, chunk, suffix) => {
        return chunk === name + suffix
      }
    },
    scrollRef: {
      type: String,
      default: ''
    },
    isFromNet: { // @数据是否从外部来
      type: Boolean,
      default: false
    }
  },
  data () {
    return {
      // at[v-model] mode should be on only when
      // initial :value/v-model is present (not nil)
      bindsValue: this.value != null,
      hasComposition: false,
      atwho: null,
      timePeriod: 200,
      temporaryStorage: [],
      chosens: []
    }
  },
  computed: {
    atItems () {
      return this.at ? [this.at] : this.ats
    },
    style () {
      if (this.atwho) {
        const { x, y } = this.atwho
        const { wrap } = this.$refs
        const el = this.$el.querySelector('textarea')
        if (wrap) {
          const left = x + el.offsetLeft - el.scrollLeft + 'px'
          const top = y + el.offsetTop - el.scrollTop + 'px'
          return { left, top }
        }
      }
      return null
    }
  },
  watch: {
    'atwho.cur' (index) {
      if (index != null) { // cur index exists
        this.$nextTick(() => {
          this.scrollToCur()
        })
      }
    },
    members () {
      this.handleInput(true)
    },
    value (value, oldValue) {
      if (this.bindsValue) {
        this.handleValueUpdate(value)
      }
    }
  },
  mounted () {
    if (this.bindsValue) {
      this.handleValueUpdate(this.value)
    }
  },
  methods: {
    itemName (v) {
      const { nameKey } = this
      return nameKey ? v[nameKey] : v
    },
    isCur (index) {
      return index === this.atwho.cur
    },

    handleItemHover (e) {
      if (this.hoverSelect) {
        this.selectByMouse(e)
      }
    },
    handleItemClick (e) {
      this.selectByMouse(e)
      this.insertItem()
    },
    handleKeyDown (e) {
      const { atwho } = this
      if (atwho) {
        if (e.keyCode === 38 || e.keyCode === 40) { // ↑/↓
          if (!(e.metaKey || e.ctrlKey)) {
            e.preventDefault()
            e.stopPropagation()
            this.selectByKeyboard(e)
          }
          return
        }
        if (e.keyCode === 13) { // enter
          this.insertItem()
          e.preventDefault()
          e.stopPropagation()
          return
        }
        if (e.keyCode === 27) { // esc
          this.closePanel()
          return
        }
      }
      // CTRL + C 时，阻止执行
      if (e.keyCode === 67 && e.ctrlKey) {
        return
      }
      // 为了兼容ie ie9~11 editable无input事件 只能靠keydown触发 textarea正常
      // 另 ie9 textarea的delete不触发input
      const isValid = (e.keyCode >= 48 && e.keyCode <= 90) || e.keyCode === 8
      if (isValid) {
        setTimeout(() => {
          this.handleInput()
        }, 50)
      }

      if (e.keyCode === 8) {
        this.handleDelete(e)
      }
    },

    // compositionStart -> input -> compositionEnd
    handleCompositionStart () {
      this.hasComposition = true
    },
    handleCompositionEnd () {
      this.hasComposition = false
      this.handleInput()
    },

    closePanel () {
      if (this.atwho) {
        this.atwho = null
      }
    },

    scrollToCur () {
      const curEl = this.$refs.cur[0]
      const scrollParent = curEl.parentElement.parentElement // .atwho-view
      scrollIntoView(curEl, scrollParent)
    },
    selectByMouse (e) {
      const el = closest(e.target, d => {
        return d.getAttribute('data-index')
      })
      const cur = +el.getAttribute('data-index')
      this.atwho = {
        ...this.atwho,
        cur
      }
    },
    selectByKeyboard (e) {
      const offset = e.keyCode === 38 ? -1 : 1
      const { cur, list } = this.atwho
      const nextCur = this.loop
        ? (cur + offset + list.length) % list.length
        : Math.max(0, Math.min(cur + offset, list.length - 1))
      this.atwho = {
        ...this.atwho,
        cur: nextCur
      }
    },
    handleValueUpdate (value) {
      const el = this.$el.querySelector('textarea')
      if (value !== el.value) { // avoid range reset
        el.value = value
      }
    },
    handleDelete (e) {
      const el = this.$el.querySelector('textarea')
      const text = el.value.slice(0, el.selectionEnd)
      if (text) {
        const { atItems, members, suffix, deleteMatch, itemName, chosens, isFromNet } = this
        const { at, index } = getAtAndIndex(text, atItems)
        if (index > -1) {
          const chunk = text.slice(index + at.length)
          let list = isFromNet ? chosens : members
          const has = list.some(v => {
            const name = itemName(v)
            return deleteMatch(name, chunk, suffix)
          })
          if (has) {
            el.value = el.value.slice(0, index) +
              el.value.slice(el.selectionEnd - 1)
            el.selectionStart = index + 1
            el.selectionEnd = index + 1
            this.handleInput()
          }
        }
      }
    },
    handleInput (keep) {
      if (this.timeout) {
        clearTimeout(this.timeout)
        this.timeout = null
      }
      this.timeout = setTimeout(() => {
        this.handleInputHandle(keep)
      }, this.timePeriod)
    },
    handleInputHandle (keep) {
      if (this.hasComposition) return
      const el = this.$el.querySelector('textarea')
      this.$emit('input', el.value)

      const text = el.value.slice(0, el.selectionEnd)
      if (text) {
        const { atItems, avoidEmail, allowSpaces, suffix } = this
        let show = true
        const { at, index } = getAtAndIndex(text, atItems) // at: @ index: @位置
        if (index < 0) show = false
        const prev = text[index - 1] // 光标前一个位置
        const chunk = text.slice(index + at.length, text.length) // @ 后面所跟字符
        if (avoidEmail) {
          // 上一个字符不能为字母数字 避免与邮箱冲突
          // 微信则是避免 所有字母数字及半角符号
          if (/^[a-z0-9]$/i.test(prev)) show = false
        }
        if (!allowSpaces && /\s/.test(chunk)) {
          show = false
        }
        // chunk以空白字符开头不匹配 避免`@ `也匹配
        if (/^\s/.test(chunk)) show = false
        if (!show) {
          this.closePanel()
        } else {
          const { members, filterMatch, itemName } = this
          if (!keep) {
            let suffixIndex = text.lastIndexOf(suffix)
            if (index > suffixIndex) {
              this.$emit('at', chunk)
            }
          }
          if (this.isFromNet) {
            this.temporaryStorage = {
              chunk, index, at, keep
            }
          } else {
            const matched = members.filter(v => {
              const name = itemName(v)
              return filterMatch(name, chunk, at)
            })
            if (matched.length) {
              this.openPanel(matched, chunk, index, at, keep)
            } else {
              this.closePanel()
            }
          }
        }
      } else {
        this.closePanel()
      }
      // let elLenth = el.value.length
      // let elEnd = el.selectionEnd
      // let step = elEnd - this.temporaryLength
      // for (let i = 0; i < this.chosens.length; i++) {
      //   let v = this.chosens[i]
      //   if (elEnd - step < v.end) {
      //     if (elEnd - step <= v.start) {
      //       v.start += step
      //     }
      //     v.end += step
      //   }
      // }
      // this.temporaryLength = elLenth
    },
    openPanel (list, chunk, offset, at) {
      const fn = () => {
        const el = this.$el.querySelector('textarea')
        const atEnd = offset + at.length // 从@后第一位开始
        const rect = getCaretCoordinates(el, atEnd)
        this.atwho = {
          chunk,
          offset,
          list,
          atEnd,
          x: rect.left,
          y: rect.top - 4,
          cur: 0 // todo: 尽可能记录
        }
      }
      if (this.atwho) {
        fn()
      } else { // 焦点超出了显示区域 需要提供延时以移动指针 再计算位置
        setTimeout(fn, 10)
      }
    },

    // todo: 抽离成库并测试
    insertText (text, ta) {
      const start = ta.selectionStart
      const end = ta.selectionEnd
      ta.value = ta.value.slice(0, start) +
        text + ta.value.slice(end)
      const newEnd = start + text.length
      ta.selectionStart = newEnd
      ta.selectionEnd = newEnd
    },
    insertItem () {
      const { list, cur, atEnd } = this.atwho
      const { suffix, atItems, itemName } = this
      const el = this.$el.querySelector('textarea')
      const text = el.value.slice(0, atEnd)
      const { at, index } = getAtAndIndex(text, atItems)
      const start = index + at.length // 从@后第一位开始
      el.selectionStart = start
      el.focus() // textarea必须focus回来
      const t = itemName(list[cur]) + suffix
      this.addChosens(list[cur])
      this.insertText(t, el)
      this.closePanel()
      this.handleInput(true)
    },
    addChosens (item) {
      this.chosens.push(item)
    },
    updateList (list) {
      const {chunk, index, at, keep} = this.temporaryStorage
      if (list.length) {
        this.openPanel(list, chunk, index, at, keep)
      } else {
        this.closePanel()
      }
    },
    test () {
      let array = []
      for (let i = 0; i < this.chosens.length; i++) {
        let v = this.chosens[i]
        let obj = {}
        obj.name = v.name
        array.push(obj)
      }
      console.table(array)
    }
  }
}
</script>
