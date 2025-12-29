<template>
  <div class="page">
    <div class="playlist-page">
      <button @click="$emit('go-back')" class="back-button">
        <i class="fas fa-arrow-left"></i> 返回首页
      </button>
      
      <div class="playlist-header">
        <div class="playlist-cover-large">
          <img :src="playlist.cover" :alt="playlist.name">
        </div>
        <div class="playlist-info-large">
          <h2>{{ playlist.name }}</h2>
          <p class="playlist-description">{{ playlist.description }}</p>
          
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
    // 计算歌单总时长
    const totalDuration = computed(() => {
      const totalSeconds = props.playlistSongs.reduce((sum, song) => sum + song.duration, 0)
      return Math.round(totalSeconds / 60)
    })
    
    // 播放全部
    const playAll = () => {
      if (props.playlistSongs.length > 0) {
        emit('play-all', props.playlist)
        emit('play-song', props.playlistSongs[0])
      }
    }
    
    // 随机播放
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