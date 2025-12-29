<template>
  <div class="player-container">
    <div class="player">
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

      <div class="progress-container">
        <span class="time-current">{{ formatTime(currentTime) }}</span>
        <div class="progress-bar" @click="handleSeek">
          <div class="progress" :style="{ width: progressPercent + '%' }"></div>
        </div>
        <span class="time-total">{{ formatTime(duration) }}</span>
      </div>

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
    // 本地状态
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

<!-- <style scoped>
.player-container {
  position: fixed;
  bottom: 0;
  left: 0;
  width: 100%;
  background: rgba(22, 33, 62, 0.95);
  backdrop-filter: blur(10px);
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  padding: 15px 20px;
  z-index: 1000;
  box-shadow: 0 -5px 20px rgba(0, 0, 0, 0.3);
}

.player {
  max-width: 1200px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
  flex-wrap: wrap;
  gap: 20px;
}

.song-info {
  display: flex;
  align-items: center;
  gap: 15px;
  flex: 1;
  min-width: 250px;
}

.album-cover {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  overflow: hidden;
  border: 3px solid #4cc9f0;
  flex-shrink: 0;
}

.album-cover.rotating {
  animation: rotate 10s linear infinite;
}

@keyframes rotate {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.album-cover img {
  width: 100%;
  height: 100%;
  object-fit: cover;
}

.song-details {
  flex: 1;
  min-width: 0;
}

.song-details h3 {
  font-size: 1.2rem;
  margin-bottom: 5px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.song-details p {
  color: rgba(255, 255, 255, 0.7);
  font-size: 0.9rem;
  margin-bottom: 3px;
}

.song-status {
  font-size: 0.8rem !important;
  color: #4cc9f0 !important;
}

.favorite-btn {
  background: none;
  border: none;
  color: #fff;
  font-size: 1.5rem;
  cursor: pointer;
  transition: all 0.3s;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
}

.favorite-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  transform: scale(1.1);
}

.playback-controls {
  display: flex;
  align-items: center;
  gap: 15px;
  flex: 1;
  justify-content: center;
}

.play-mode-btn, .control-btn, .play-btn {
  background: rgba(255, 255, 255, 0.1);
  border: none;
  color: #fff;
  width: 50px;
  height: 50px;
  border-radius: 50%;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.2rem;
  transition: all 0.3s;
}

.play-btn {
  width: 60px;
  height: 60px;
  background: #4cc9f0;
  font-size: 1.5rem;
}

/* 禁用按钮样式 */
.play-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.play-btn:disabled:hover {
  background: #4cc9f0;
  transform: none;
}

.play-mode-btn:hover, .control-btn:hover {
  background: rgba(255, 255, 255, 0.2);
}

.play-btn:not(:disabled):hover {
  background: #3ab5db;
  transform: scale(1.05);
}

.progress-container {
  display: flex;
  align-items: center;
  gap: 15px;
  flex: 2;
  min-width: 300px;
}

.time-current, .time-total {
  font-size: 0.9rem;
  color: rgba(255, 255, 255, 0.7);
  min-width: 50px;
}

.progress-bar {
  flex: 1;
  height: 6px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 3px;
  cursor: pointer;
  position: relative;
  transition: height 0.2s;
}

.progress-bar:hover {
  height: 8px;
}

.progress {
  height: 100%;
  background: #4cc9f0;
  border-radius: 3px;
  transition: width 0.1s;
}

.volume-container {
  display: flex;
  align-items: center;
  gap: 10px;
  flex: 1;
  min-width: 150px;
}

.volume-icon-btn {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.7);
  font-size: 1.2rem;
  cursor: pointer;
  width: 30px;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: color 0.3s;
}

.volume-icon-btn:hover {
  color: #fff;
}

.volume-bar {
  flex: 1;
  height: 6px;
  background: rgba(255, 255, 255, 0.2);
  border-radius: 3px;
  cursor: pointer;
  transition: height 0.2s;
}

.volume-bar:hover {
  height: 8px;
}

.volume-level {
  height: 100%;
  background: #4cc9f0;
  border-radius: 3px;
  transition: width 0.3s;
}

/* 响应式设计 */
@media (max-width: 992px) {
  .player {
    flex-direction: column;
    gap: 15px;
  }
  
  .song-info, .playback-controls, .progress-container, .volume-container {
    width: 100%;
  }
  
  .progress-container {
    min-width: 100%;
  }
}

@media (max-width: 768px) {
  .player-container {
    padding: 10px 15px;
  }
  
  .song-details h3 {
    font-size: 1rem;
  }
  
  .song-details p {
    font-size: 0.8rem;
  }
  
  .album-cover {
    width: 50px;
    height: 50px;
  }
  
  .play-mode-btn, .control-btn {
    width: 40px;
    height: 40px;
    font-size: 1rem;
  }
  
  .play-btn {
    width: 50px;
    height: 50px;
    font-size: 1.2rem;
  }
  
  .progress-container {
    min-width: 100%;
  }
  
  .time-current, .time-total {
    font-size: 0.8rem;
    min-width: 40px;
  }
}

/* 加载动画 */
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.fa-spin {
  animation: spin 1s linear infinite;
}
</style> -->