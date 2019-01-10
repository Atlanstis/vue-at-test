<style lang="scss" src="./At.scss"></style>

<template>
  <div ref="wrap"
    class="atwho__wrap"
    @compositionstart="handleCompositionStart"
    @compositionend="handleCompositionEnd"
    @input="handleInput()"
    @keydown="handleKeyDown"
  >
    <div v-if="atwho"
      class="atwho__panel"
      :style="style"
    >
      <div class="atwho__inner">
        <div class="atwho__view">
          <ul class="atwho__ul">
            <li v-for="(item, index) in atwho.list"
              :key="index"
              class="atwho__li"
              :class="isCur(index) && 'atwho__cur'"
              :ref="isCur(index) && 'cur'"
              :data-index="index"
              @mouseenter="handleItemHover"
              @click="handleItemClick"
            >
              <slot name="item" :item="item">
                <span v-text="itemName(item)"></span>
              </slot>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <slot></slot>
  </div>
</template>
