<template>
  <!-- 导航栏 -->
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
  // 组件的属性：当前页面（依赖父组件传递）
  props: {
    currentPage: {
      type: String,
      required: true
    }
  },
  // 声明当前组件要触发的自定义事件（向父组件传递消息）
  emits: ['change-page']
}
</script>