<template>
  <div class="page">
    <div class="library-page">
      <h2><i class="fas fa-compact-disc"></i> 音乐库</h2>
      <p class="page-description">浏览所有可用的歌曲。点击任意歌曲开始播放。</p>
      
      <div class="library-stats">
        <div class="stat-card">
          <i class="fas fa-music"></i>
          <div>
            <h4>歌曲总数</h4>
            <p>{{ songs.length }} 首</p>
          </div>
        </div>
        <div class="stat-card">
          <i class="fas fa-heart"></i>
          <div>
            <h4>已收藏</h4>
            <p>{{ favorites.length }} 首</p>
          </div>
        </div>
        <div class="stat-card">
          <i class="fas fa-search"></i>
          <div>
            <h4>搜索结果</h4>
            <p>{{ filteredSongs.length }} 首</p>
          </div>
        </div>
      </div>
      
      <FilterBar 
        :genres="genres"
        :currentGenre="currentGenre"
        :searchQuery="searchQuery"
        @filter-by-genre="$emit('filter-by-genre', $event)"
        @update-search="$emit('update-search', $event)"
        @clear-search="$emit('clear-search')"
      />
      
      <div v-if="filteredSongs.length > 0" class="song-list">
        <SongItem 
          v-for="song in filteredSongs" 
          :key="song.id"
          :song="song"
          :isCurrent="currentSong && currentSong.id === song.id"
          :isPlaying="isPlaying"
          :isFavorite="favorites.includes(song.id)"
          @play-song="$emit('play-song', song)"
          @toggle-favorite="$emit('toggle-favorite', song.id)"
        />
      </div>
      
      <div v-else class="empty-state">
        <i class="fas fa-search"></i>
        <h3>未找到相关歌曲</h3>
        <p>尝试更改筛选条件或搜索关键词</p>
        <button @click="$emit('clear-search')" class="filter-btn" style="margin-top: 15px;">
          清除搜索
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import SongItem from '../components/SongItem.vue'
import FilterBar from '../components/FilterBar.vue'

export default {
  name: 'LibraryPage',
  components: {
    SongItem,
    FilterBar
  },
  props: {
    songs: {
      type: Array,
      required: true
    },
    genres: {
      type: Array,
      required: true
    },
    currentGenre: {
      type: String,
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
    searchQuery: {
      type: String,
      default: ''
    },
    filteredSongs: {
      type: Array,
      required: true
    }
  },
  emits: ['play-song', 'toggle-favorite', 'filter-by-genre', 'update-search', 'clear-search']
}
</script>

<style scoped>
.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: rgba(255, 255, 255, 0.6);
}

.empty-state i {
  font-size: 4rem;
  margin-bottom: 20px;
  color: #4cc9f0;
}

.empty-state h3 {
  font-size: 1.8rem;
  margin-bottom: 10px;
  color: rgba(255, 255, 255, 0.8);
}

.empty-state p {
  font-size: 1.1rem;
  max-width: 400px;
  margin: 0 auto;
  line-height: 1.6;
}

.library-stats {
  display: flex;
  gap: 20px;
  margin-bottom: 30px;
  flex-wrap: wrap;
}

.stat-card {
  background: rgba(255, 255, 255, 0.08);
  padding: 20px;
  border-radius: 15px;
  flex: 1;
  min-width: 200px;
  display: flex;
  align-items: center;
  gap: 20px;
}

.stat-card i {
  font-size: 2.5rem;
  color: #4cc9f0;
}

.stat-card h4 {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 5px;
}

.stat-card p {
  font-size: 1.5rem;
  font-weight: bold;
  color: #fff;
}

@media (max-width: 768px) {
  .library-stats {
    flex-direction: column;
  }
  
  .stat-card {
    width: 100%;
  }
}
</style>