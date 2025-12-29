<template>
  <header>
    <h1><i class="fas fa-music"></i> 音乐播放站</h1>
    <nav>
      <button @click="$emit('change-page', 'home')" :class="{active: currentPage === 'home'}">
        <i class="fas fa-home"></i> 首页
      </button>
      <button @click="$emit('change-page', 'library')" :class="{active: currentPage === 'library'}">
        <i class="fas fa-compact-disc"></i> 音乐库
      </button>
      <button @click="$emit('change-page', 'my')" :class="{active: currentPage === 'my'}">
        <i class="fas fa-user"></i> 我的页面
      </button>
    </nav>
  </header>
</template>

<script>
export default {
  name: 'Header',
  props: {
    currentPage: {
      type: String,
      required: true
    }
  },
  emits: ['change-page']
}
</script>