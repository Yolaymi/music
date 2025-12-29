<template>
  <div class="page">
    <div class="my-page">
      <h2><i class="fas fa-user"></i> 我的页面</h2>
      <p class="page-description">管理你的收藏和查看播放历史</p>
      
      <div class="my-page-sections">
        <div class="my-page-section">
          <h3><i class="fas fa-heart"></i> 我的收藏</h3>
          
          <div v-if="favoriteSongs.length === 0" class="empty-state" style="padding: 30px 20px;">
            <i class="fas fa-heart-broken"></i>
            <h3>暂无收藏的歌曲</h3>
            <p>去音乐库发现你喜欢的音乐并收藏它们吧！</p>
          </div>
          
          <div v-else>
            <div class="song-list">
              <SongItem 
                v-for="song in favoriteSongs" 
                :key="song.id"
                :song="song"
                :isCurrent="currentSong && currentSong.id === song.id"
                :isPlaying="isPlaying"
                :isFavorite="true"
                @play-song="$emit('play-song', song)"
                @toggle-favorite="$emit('toggle-favorite', song.id)"
              />
            </div>
          </div>
        </div>
        
        <div class="my-page-section">
          <h3><i class="fas fa-history"></i> 最近播放</h3>
          
          <div v-if="recentlyPlayed.length === 0" class="empty-state" style="padding: 30px 20px;">
            <i class="fas fa-clock"></i>
            <h3>暂无播放记录</h3>
            <p>开始播放音乐，这里会显示你的播放历史</p>
          </div>
          
          <div v-else>
            <div class="song-list">
              <div 
                v-for="song in recentlyPlayed" 
                :key="song.id"
                class="song-item"
                :class="{ playing: currentSong && currentSong.id === song.id }"
                @click="$emit('play-song', song)"
              >
                <div class="album-cover">
                  <img :src="song.cover" :alt="song.name">
                </div>
                <div class="song-details">
                  <h3>{{ song.name }}</h3>
                  <p>{{ song.artist }}</p>
                </div>
                <div class="song-actions">
                  <button @click="$emit('play-song', song)" class="play-btn-small">
                    <i class="fas" :class="currentSong && currentSong.id === song.id && isPlaying ? 'fa-pause' : 'fa-play'"></i>
                  </button>
                  <button @click="$emit('toggle-favorite', song.id)" class="favorite-btn">
                    <i class="fas" :class="favorites.includes(song.id) ? 'fa-heart' : 'fa-heart-o'"></i>
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import SongItem from '../components/SongItem.vue'

export default {
  name: 'MyPage',
  components: {
    SongItem
  },
  props: {
    favoriteSongs: {
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
    },
    recentlyPlayed: {
      type: Array,
      required: true
    }
  },
  emits: ['play-song', 'toggle-favorite']
}
</script>