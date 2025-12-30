<template>
  <div 
    class="song-item"
    :class="{ playing: isCurrent && isPlaying }"
  >
    <!-- 专辑封面区域 -->
    <div class="album-cover">
      <img :src="song.cover" :alt="song.name">
    </div>
    <!-- 歌曲详情区域：点击时触发播放事件 -->
    <div class="song-details" @click="$emit('play-song', song)">
      <h3>{{ song.name }}</h3>
      <p>{{ song.artist }} • {{ formatDuration(song.duration) }}</p>
    </div>
    <!-- 歌曲操作按钮区域（播放/收藏） -->
    <div class="song-actions">
      <!-- 播放/暂停按钮：点击触发play-song事件，传递当前歌曲数据 -->
      <button @click="$emit('play-song', song)" class="play-btn-small">
        <i class="fas" :class="isCurrent && isPlaying ? 'fa-pause' : 'fa-play'"></i>
      </button>
      <!-- 收藏/取消收藏按钮：点击触发toggle-favorite事件，传递当前歌曲id -->
      <button @click="$emit('toggle-favorite', song.id)" class="favorite-btn">
        <i class="fas" :class="isFavorite ? 'fa-heart' : 'fa-heart-o'"></i>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SongItem',
   // 接收父组件传递的props数据：当前歌曲、是否当前播放、是否播放中、是否收藏
  props: {
    song: {
      type: Object,
      required: true
    },
    isCurrent: {
      type: Boolean,
      default: false
    },
    isPlaying: {
      type: Boolean,
      default: false
    },
    isFavorite: {
      type: Boolean,
      default: false
    }
  },
  // 声明当前组件要触发的自定义事件（向父组件传递交互消息）
  emits: ['play-song', 'toggle-favorite'],
  // 时长格式化方法：将“秒数”转换为“分:秒”格式（如125秒 → 2:05）
  setup() {
    const formatDuration = (seconds) => {
      const mins = Math.floor(seconds / 60)
      const secs = seconds % 60
      return `${mins}:${secs.toString().padStart(2, '0')}`
    }
    
    return {
      formatDuration
    }
  }
}
</script>