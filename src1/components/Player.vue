<template>
  <!-- 播放器容器 -->
  <div class="player-container">
    <div class="player">
      <!-- 1. 歌曲信息区域：展示封面、歌名、歌手、收藏状态、加载状态 -->
      <div class="song-info">
        <div class="album-cover" :class="{ rotating: isPlaying }">
          <img :src="currentSong.cover" :alt="currentSong.name">
        </div>
        <div class="song-details">
          <h3>{{ currentSong.name }}</h3>
          <p>{{ currentSong.artist }}</p>
          <!-- 只在确实加载中且未播放时显示 -->
          <p class="song-status" v-if="isLoading && !isPlaying">
            <i class="fas fa-spinner fa-spin"></i> 加载中...
          </p>
        </div>
        <button 
          class="favorite-btn" 
          @click="$emit('toggle-favorite', currentSong.id)"
          :title="favorites.includes(currentSong.id) ? '取消收藏' : '收藏'"
        >
          <i class="fas fa-heart" :style="{color: favorites.includes(currentSong.id) ? '#ff4757' : '#fff'}"></i>
        </button>
      </div>

      <!-- 2. 播放控制区域：播放模式、上一首、播放/暂停、下一首 -->
      <div class="playback-controls">
        <button 
          @click="$emit('change-play-mode')" 
          class="play-mode-btn" 
          :title="playModeText"
        >
          <i class="fas" :class="playModeIcon"></i>
        </button>
        <button 
          @click="$emit('previous-song')" 
          class="control-btn"
          :title="'上一首'"
        >
          <i class="fas fa-step-backward"></i>
        </button>
        <button 
          @click="$emit('toggle-play')" 
          class="play-btn"
          :title="isPlaying ? '暂停' : '播放'"
          :disabled="isLoading && !isPlaying"
        >
          <!-- 简化图标逻辑 -->
          <i v-if="isLoading && !isPlaying" class="fas fa-spinner fa-spin"></i>
          <i v-else class="fas" :class="isPlaying ? 'fa-pause' : 'fa-play'"></i>
        </button>
        <button 
          @click="$emit('next-song')" 
          class="control-btn"
          :title="'下一首'"
        >
          <i class="fas fa-step-forward"></i>
        </button>
      </div>

      <!-- 3. 进度条区域：当前播放时间、进度条（可点击跳转）、总时长 -->
      <div class="progress-container">
        <span class="time-current">{{ formatTime(currentTime) }}</span>
        <div class="progress-bar" @click="handleSeek">
          <div class="progress" :style="{ width: progressPercent + '%' }"></div>
        </div>
        <span class="time-total">{{ formatTime(duration) }}</span>
      </div>

      <!-- 4. 音量控制区域：静音按钮、音量条（可点击调整） -->
      <div class="volume-container">
        <button class="volume-icon-btn" @click="toggleMute" :title="isMuted ? '取消静音' : '静音'">
          <i class="fas" :class="volumeIcon"></i>
        </button>
        <div class="volume-bar" @click="handleVolumeAdjust">
          <div class="volume-level" :style="{ width: (isMuted ? 0 : volume) * 100 + '%' }"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue'

export default {
  name: 'Player',
  props: {
    currentSong: {
      type: Object,
      required: true
    },
    isPlaying: {
      type: Boolean,
      required: true
    },
    currentTime: {
      type: Number,
      required: true
    },
    duration: {
      type: Number,
      required: true
    },
    progressPercent: {
      type: Number,
      required: true
    },
    playMode: {
      type: Number,
      required: true
    },
    volume: {
      type: Number,
      required: true
    },
    favorites: {
      type: Array,
      required: true
    },
    isLoading: {
      type: Boolean,
      default: false
    }
  },
  emits: ['toggle-play', 'next-song', 'previous-song', 'change-play-mode', 'toggle-favorite', 'seek-to', 'adjust-volume'],
  setup(props, { emit }) {
    // 本地状态：是否静音（组件内部维护，不依赖父组件）
    const isMuted = ref(false)
    
    // 播放模式文本
    const playModeText = computed(() => {
      const modes = ['顺序播放', '单曲循环', '随机播放']
      return modes[props.playMode]
    })
    
    // 播放模式图标
    const playModeIcon = computed(() => {
      const icons = ['fa-list-ol', 'fa-repeat', 'fa-random']
      return icons[props.playMode]
    })
    
    // 音量图标
    const volumeIcon = computed(() => {
      if (props.volume === 0 || isMuted.value) {
        return 'fa-volume-mute'
      } else if (props.volume < 0.5) {
        return 'fa-volume-down'
      } else {
        return 'fa-volume-up'
      }
    })
    
    // 格式化时间（秒 → 分:秒）
    const formatTime = (seconds) => {
      if (!seconds && seconds !== 0) return '0:00'
      const mins = Math.floor(seconds / 60)
      const secs = Math.floor(seconds % 60)
      return `${mins}:${secs.toString().padStart(2, '0')}`
    }
    
    // 处理进度条点击
    const handleSeek = (event) => {
      emit('seek-to', event)
    }
    
    // 处理音量调整
    const handleVolumeAdjust = (event) => {
      emit('adjust-volume', event)
      if (props.volume > 0 && isMuted.value) {
        isMuted.value = false
      }
    }
    
    // 切换静音
    const toggleMute = () => {
      isMuted.value = !isMuted.value
      if (!isMuted.value && props.volume === 0) {
        // 发送一个事件来调整音量到默认值
        const volumeBar = document.createElement('div')
        volumeBar.style.width = '100px'
        volumeBar.style.height = '10px'
        const clickEvent = {
          currentTarget: volumeBar,
          offsetX: 70 // 70% 音量
        }
        emit('adjust-volume', clickEvent)
      } else if (isMuted.value) {
        // 静音时发送0音量
        const volumeBar = document.createElement('div')
        volumeBar.style.width = '100px'
        volumeBar.style.height = '10px'
        const clickEvent = {
          currentTarget: volumeBar,
          offsetX: 0 // 0% 音量
        }
        emit('adjust-volume', clickEvent)
      }
    }
    
    // 监听音量变化
    watch(() => props.volume, (newVolume) => {
      if (newVolume > 0 && isMuted.value) {
        isMuted.value = false
      }
    })
    
    return {
      playModeText,
      playModeIcon,
      volumeIcon,
      isMuted,
      formatTime,
      handleSeek,
      handleVolumeAdjust,
      toggleMute
    }
  }
}
</script>
