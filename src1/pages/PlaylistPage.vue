<template>
  <div class="page">
    <div class="playlist-page">
      <button @click="$emit('go-back')" class="back-button">
        <i class="fas fa-arrow-left"></i> 返回首页
      </button>
      <!-- 歌单头部：展示歌单封面、名称、描述、统计信息、操作按钮 -->
      <div class="playlist-header">
        <!-- 歌单大封面 -->
        <div class="playlist-cover-large">
          <img :src="playlist.cover" :alt="playlist.name">
        </div>
        <!-- 歌单详细信息区 -->
        <div class="playlist-info-large">
          <h2>{{ playlist.name }}</h2>
          <p class="playlist-description">{{ playlist.description }}</p>
          
          <!-- 歌单统计信息：歌曲数量、总时长 -->
          <div class="playlist-stats">
            <div class="playlist-stat">
              <i class="fas fa-music"></i>
              <span>{{ playlistSongs.length }} 首歌曲</span>
            </div>
            <div class="playlist-stat">
              <i class="fas fa-clock"></i>
              <span>{{ totalDuration }} 分钟</span>
            </div>
          </div>
          
          <!-- 歌单操作按钮：播放全部、随机播放 -->
          <div class="playlist-actions">
            <button @click="playAll" class="playlist-play-btn">
              <i class="fas fa-play"></i> 播放全部
            </button>
            <button @click="shufflePlay" class="playlist-play-btn">
              <i class="fas fa-random"></i> 随机播放
            </button>
          </div>
        </div>
      </div>
      
      <!-- 歌单内歌曲列表：复用SongItem组件渲染每首歌曲 -->
      <div class="song-list">
        <SongItem 
          v-for="song in playlistSongs" 
          :key="song.id"
          :song="song"
          :isCurrent="currentSong && currentSong.id === song.id"
          :isPlaying="isPlaying"
          :isFavorite="favorites.includes(song.id)"
          @play-song="$emit('play-song', song)"
          @toggle-favorite="$emit('toggle-favorite', song.id)"
        />
      </div>
    </div>
  </div>
</template>

<script>
import { computed } from 'vue'
import SongItem from '../components/SongItem.vue'

export default {
  name: 'PlaylistPage',
  components: {
    SongItem
  },
  props: {
    playlist: {
      type: Object,
      required: true
    },
    playlistSongs: {
      type: Array,
      required: true
    },
    currentSong: {
      type: Object,
      default: null
    },
    isPlaying: {
      type: Boolean,
      required: true
    },
    favorites: {
      type: Array,
      required: true
    }
  },
  emits: ['play-song', 'toggle-favorite', 'go-back', 'play-all'],
  setup(props, { emit }) {
    // 计算歌单总时长（将所有歌曲的秒数转换为总分钟数，四舍五入）
    const totalDuration = computed(() => {
      const totalSeconds = props.playlistSongs.reduce((sum, song) => sum + song.duration, 0)
      return Math.round(totalSeconds / 60)
    })
    
    // 播放全部：按歌单顺序播放所有歌曲（先播放第一首）
    const playAll = () => {
      if (props.playlistSongs.length > 0) {
        emit('play-all', props.playlist)
        emit('play-song', props.playlistSongs[0])
      }
    }
    
    // 随机播放：从歌单内随机选一首歌曲播放
    const shufflePlay = () => {
      if (props.playlistSongs.length > 0) {
        const randomIndex = Math.floor(Math.random() * props.playlistSongs.length)
        emit('play-song', props.playlistSongs[randomIndex])
      }
    }
    
    return {
      totalDuration,
      playAll,
      shufflePlay
    }
  }
}
</script>