<template>
  <div class="page">
    <div class="home-page">
      <div class="welcome-section">
        <h2>欢迎来到音乐播放站</h2>
        <p>在这里发现和聆听你最喜欢的音乐。我们为你精选了各种风格的歌曲，从流行到摇滚，从古典到电子，总有一首适合你。</p>
        <p>当前播放: {{ currentSong ? currentSong.name : '暂无' }}</p>
        <button @click="$emit('toggle-play')" class="play-btn-large">
          <i class="fas" :class="isPlaying ? 'fa-pause' : 'fa-play'"></i>
          {{ isPlaying ? '暂停播放' : '开始播放' }}
        </button>
      </div>
      
      <div class="featured-playlists-section">
        <h3><i class="fas fa-star"></i> 精选歌单</h3>
        <div class="playlists-container">
          <button class="scroll-btn left" @click="scrollLeft">
            <i class="fas fa-chevron-left"></i>
          </button>
          <div class="playlists-scroll" ref="playlistsScroll">
            <PlaylistCard 
              v-for="playlist in playlists" 
              :key="playlist.id"
              :playlist="playlist"
              @view-playlist="$emit('view-playlist', playlist)"
            />
          </div>
          <button class="scroll-btn right" @click="scrollRight">
            <i class="fas fa-chevron-right"></i>
          </button>
        </div>
      </div>
      
      <div class="featured-playlist">
        <h3><i class="fas fa-fire"></i> 热门推荐</h3>
        <div class="song-list">
          <div 
            v-for="song in songs.slice(0, 4)" 
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
              <button class="play-btn-small">
                <i class="fas" :class="currentSong && currentSong.id === song.id && isPlaying ? 'fa-pause' : 'fa-play'"></i>
              </button>
            </div>
          </div>
        </div>
      </div>
      
      <div class="quick-links">
        <h3><i class="fas fa-bolt"></i> 快速访问</h3>
        <div class="links">
          <!-- 立即播放卡片 -->
          <div 
            class="link-card" 
            :class="{ clicked: isPlaying && currentSong && currentSong.id === lastPlayedSong?.id }"
            @click="handleQuickPlay"
            @mouseleave="resetClickState('quickPlay')"
          >
            <div class="hot-badge" v-if="mostPlayedSongs.length > 0">
              推荐
            </div>
            <div class="tooltip">
              {{ getQuickPlayTooltip() }}
            </div>
            <div class="playing-indicator" v-if="isPlaying && currentSong && currentSong.id === lastPlayedSong?.id"></div>
            <i class="fas fa-play-circle"></i>
            <span>立即播放</span>
            <div class="recommendation-reason">
              {{ getQuickPlayReason() }}
            </div>
          </div>
          
          <!-- 热门单曲卡片 -->
          <div 
            class="link-card" 
            :class="{ clicked: isHotCardClicked }"
            @click="handleHotSongPlay"
            @mouseleave="resetClickState('hotSong')"
          >
            <div class="hot-badge">
              热
            </div>
            <div class="tooltip">
              播放热门歌曲：{{ getHotSongName() }}
            </div>
            <div class="playing-indicator" v-if="isPlaying && currentSong && currentSong.id === getHotSongId()"></div>
            <i class="fas fa-fire"></i>
            <span>热门单曲</span>
            <div class="recommendation-reason">
              播放次数：{{ getHotSongPlayCount() }}次
            </div>
          </div>
          
          <!-- 推荐新歌卡片 -->
          <div 
            class="link-card" 
            :class="{ clicked: isNewCardClicked }"
            @click="handleNewSongPlay"
            @mouseleave="resetClickState('newSong')"
          >
            <div class="new-badge">
              新
            </div>
            <div class="tooltip">
              发现新音乐：{{ getNewSongName() }}
            </div>
            <div class="playing-indicator" v-if="isPlaying && currentSong && currentSong.id === getNewSongId()"></div>
            <i class="fas fa-magic"></i>
            <span>推荐新歌</span>
            <div class="recommendation-reason">
              你可能还没听过
            </div>
          </div>
        </div>
      </div>
      
      <!-- 页脚 -->
      <footer class="container">
        <p>享受音乐，享受生活</p>
        <p>本网站仅用于学习展示，歌曲资源来源于网络</p>
      </footer>

    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import PlaylistCard from '../components/PlaylistCard.vue'

export default {
  name: 'HomePage',
  components: {
    PlaylistCard
  },
  props: {
    songs: {
      type: Array,
      required: true
    },
    playlists: {
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
    lastPlayedSong: {
      type: Object,
      default: null
    },
    mostPlayedSongs: {
      type: Array,
      required: true
    },
    unplayedSongs: {
      type: Array,
      required: true
    }
  },
  emits: ['play-song', 'view-playlist', 'toggle-play'],
  setup(props, { emit }) {
    const playlistsScroll = ref(null)
    const isHotCardClicked = ref(false)
    const isNewCardClicked = ref(false)
    
    // 选择的随机热门歌曲
    const selectedHotSong = ref(null)
    // 选择的随机新歌
    const selectedNewSong = ref(null)
    
    // 初始化时选择随机的热门歌曲和新歌
    onMounted(() => {
      if (props.mostPlayedSongs.length > 0) {
        const randomIndex = Math.floor(Math.random() * Math.min(3, props.mostPlayedSongs.length))
        selectedHotSong.value = props.mostPlayedSongs[randomIndex]
      }
      
      if (props.unplayedSongs.length > 0) {
        const randomIndex = Math.floor(Math.random() * Math.min(3, props.unplayedSongs.length))
        selectedNewSong.value = props.unplayedSongs[randomIndex]
      }
    })
    
    const scrollLeft = () => {
      if (playlistsScroll.value) {
        playlistsScroll.value.scrollBy({ left: -300, behavior: 'smooth' })
      }
    }
    
    const scrollRight = () => {
      if (playlistsScroll.value) {
        playlistsScroll.value.scrollBy({ left: 300, behavior: 'smooth' })
      }
    }
    
    // 立即播放处理
    const handleQuickPlay = () => {
      // 播放上次播放的歌曲或随机一首
      if (props.lastPlayedSong) {
        emit('play-song', props.lastPlayedSong)
      } else if (props.songs.length > 0) {
        const randomSong = props.songs[Math.floor(Math.random() * props.songs.length)]
        emit('play-song', randomSong)
      }
    }
    
    // 播放热门歌曲处理
    const handleHotSongPlay = () => {
      isHotCardClicked.value = true
      
      // 如果已经选择了热门歌曲，播放它
      if (selectedHotSong.value) {
        emit('play-song', selectedHotSong.value)
      } else if (props.mostPlayedSongs.length > 0) {
        // 否则随机选择一首热门歌曲
        const randomIndex = Math.floor(Math.random() * Math.min(3, props.mostPlayedSongs.length))
        selectedHotSong.value = props.mostPlayedSongs[randomIndex]
        emit('play-song', selectedHotSong.value)
      } else if (props.songs.length > 0) {
        // 如果没有热门歌曲，随机播放一首
        const randomSong = props.songs[Math.floor(Math.random() * props.songs.length)]
        emit('play-song', randomSong)
      }
    }
    
    // 播放新歌处理
    const handleNewSongPlay = () => {
      isNewCardClicked.value = true
      
      // 如果已经选择了新歌，播放它
      if (selectedNewSong.value) {
        emit('play-song', selectedNewSong.value)
      } else if (props.unplayedSongs.length > 0) {
        // 否则随机选择一首新歌
        const randomIndex = Math.floor(Math.random() * Math.min(3, props.unplayedSongs.length))
        selectedNewSong.value = props.unplayedSongs[randomIndex]
        emit('play-song', selectedNewSong.value)
      } else if (props.songs.length > 0) {
        // 如果没有新歌，随机播放一首播放次数少的
        const leastPlayed = [...props.songs].sort((a, b) => a.playCount - b.playCount)[0]
        emit('play-song', leastPlayed)
      }
    }
    
    // 重置点击状态
    const resetClickState = (type) => {
      setTimeout(() => {
        if (type === 'hotSong') {
          isHotCardClicked.value = false
        } else if (type === 'newSong') {
          isNewCardClicked.value = false
        }
      }, 300)
    }
    
    // 获取热门歌曲名称
    const getHotSongName = () => {
      return selectedHotSong.value ? selectedHotSong.value.name : 
             (props.mostPlayedSongs.length > 0 ? props.mostPlayedSongs[0].name : '随机歌曲')
    }
    
    // 获取热门歌曲ID
    const getHotSongId = () => {
      return selectedHotSong.value ? selectedHotSong.value.id : null
    }
    
    // 获取热门歌曲播放次数
    const getHotSongPlayCount = () => {
      return selectedHotSong.value ? selectedHotSong.value.playCount : 
             (props.mostPlayedSongs.length > 0 ? props.mostPlayedSongs[0].playCount : 0)
    }
    
    // 获取新歌名称
    const getNewSongName = () => {
      return selectedNewSong.value ? selectedNewSong.value.name : 
             (props.unplayedSongs.length > 0 ? props.unplayedSongs[0].name : '新发现')
    }
    
    // 获取新歌ID
    const getNewSongId = () => {
      return selectedNewSong.value ? selectedNewSong.value.id : null
    }
    
    // 获取立即播放提示
    const getQuickPlayTooltip = () => {
      if (props.lastPlayedSong) {
        return `继续播放：${props.lastPlayedSong.name}`
      }
      return '播放随机歌曲'
    }
    
    // 获取立即播放推荐理由
    const getQuickPlayReason = () => {
      if (props.lastPlayedSong) {
        return `继续上次的播放`
      }
      return '随机播放一首好歌'
    }
    
    return {
      playlistsScroll,
      isHotCardClicked,
      isNewCardClicked,
      selectedHotSong,
      selectedNewSong,
      scrollLeft,
      scrollRight,
      handleQuickPlay,
      handleHotSongPlay,
      handleNewSongPlay,
      resetClickState,
      getHotSongName,
      getHotSongId,
      getHotSongPlayCount,
      getNewSongName,
      getNewSongId,
      getQuickPlayTooltip,
      getQuickPlayReason
    }
  }
}
</script>