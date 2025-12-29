<template>
  <div id="app">
    <Header 
      :currentPage="currentPage" 
      @change-page="changePage"
      v-if="currentPage !== 'playlist'"
    />
    
    <main>
      <!-- 首页 -->
      <HomePage 
        v-if="currentPage === 'home'"
        :songs="songs"
        :playlists="playlists"
        :currentSong="currentSong"
        :isPlaying="isPlaying"
        :lastPlayedSong="lastPlayedSong"
        :mostPlayedSongs="mostPlayedSongs"
        :unplayedSongs="unplayedSongs"
        :favorites="favorites"
        @play-song="playSong"
        @view-playlist="viewPlaylist"
        @toggle-play="togglePlay"
        @play-random-hot="playRandomHotSong"
        @play-recommended-new="playRecommendedNewSong"
      />
      
      <!-- 音乐库 -->
      <LibraryPage 
        v-if="currentPage === 'library'"
        :songs="songs"
        :genres="genres"
        :currentGenre="currentGenre"
        :currentSong="currentSong"
        :isPlaying="isPlaying"
        :favorites="favorites"
        :searchQuery="searchQuery"
        :filteredSongs="filteredSongs"
        @play-song="playSong"
        @toggle-favorite="toggleFavorite"
        @filter-by-genre="filterByGenre"
        @update-search="updateSearch"
        @clear-search="clearSearch"
      />
      
      <!-- 我的页面 -->
      <MyPage 
        v-if="currentPage === 'my'"
        :favoriteSongs="favoriteSongs"
        :currentSong="currentSong"
        :isPlaying="isPlaying"
        :favorites="favorites"
        :recentlyPlayed="recentlyPlayed"
        @play-song="playSong"
        @toggle-favorite="toggleFavorite"
        @play-last-played="playLastPlayedOrRandom"
      />
      
      <!-- 歌单页面 -->
      <PlaylistPage 
        v-if="currentPage === 'playlist' && currentPlaylist"
        :playlist="currentPlaylist"
        :playlistSongs="playlistSongs"
        :currentSong="currentSong"
        :isPlaying="isPlaying"
        :favorites="favorites"
        @play-song="playSong"
        @toggle-favorite="toggleFavorite"
        @go-back="goBack"
        @play-all="playAllSongs"
      />
    </main>

    <!-- 播放器控件 -->
    <Player 
      v-if="currentSong"
      :currentSong="currentSong"
      :isPlaying="isPlaying"
      :currentTime="currentTime"
      :duration="duration"
      :progressPercent="progressPercent"
      :playMode="playMode"
      :volume="volume"
      :favorites="favorites"
      :playModeText="playModeText"
      :playModeIcon="playModeIcon"
      :isLoading="isLoading"
      @toggle-play="togglePlay"
      @next-song="nextSong"
      @previous-song="previousSong"
      @change-play-mode="changePlayMode"
      @toggle-favorite="toggleFavorite"
      @seek-to="seekTo"
      @adjust-volume="adjustVolume"
    />
    
    <!-- 隐藏的音频元素 -->
    <audio 
      ref="audioElement"
      @timeupdate="handleTimeUpdate"
      @loadedmetadata="handleLoadedMetadata"
      @ended="handleSongEnded"
      @error="handleAudioError"
      preload="auto"
    />
  </div>
</template>

<script>
import { ref, computed, onMounted, watch } from 'vue'
import Header from './components/Header.vue'
import Player from './components/Player.vue'
import HomePage from './pages/HomePage.vue'
import LibraryPage from './pages/LibraryPage.vue'
import MyPage from './pages/MyPage.vue'
import PlaylistPage from './pages/PlaylistPage.vue'

export default {
  name: 'App',
  components: {
    Header,
    Player,
    HomePage,
    LibraryPage,
    MyPage,
    PlaylistPage
  },
  setup() {
    // 当前页面
    const currentPage = ref('home')
    
    // 播放状态
    const isPlaying = ref(false)
    const currentSong = ref(null)
    const currentTime = ref(0)
    const duration = ref(0)
    const volume = ref(0.7)
    const isLoading = ref(false)
    
    // 播放模式：0-顺序播放，1-单曲循环，2-随机播放
    const playMode = ref(0)
    const playModeText = computed(() => {
      const modes = ['顺序播放', '单曲循环', '随机播放']
      return modes[playMode.value]
    })
    
    const playModeIcon = computed(() => {
      const icons = ['fa-list-ol', 'fa-repeat', 'fa-random']
      return icons[playMode.value]
    })
    
    // 收藏列表
    const favorites = ref([1, 3, 5])
    
    // 最近播放列表
    const recentlyPlayed = ref([])
    
    // 歌曲播放次数统计
    const playCounts = ref({})
    
    // 最后播放的歌曲
    const lastPlayedSong = ref(null)
    
    // 当前筛选的歌曲风格
    const currentGenre = ref('all')
    
    // 搜索查询
    const searchQuery = ref('')
    
    // 当前查看的歌单
    const currentPlaylist = ref(null)
    
    // 用于随机播放的已播歌曲记录
    const playedInRandomMode = ref([])
    
    // 音频元素引用
    const audioElement = ref(null)
    
    // 歌曲数据 - 添加音频文件URL
    const songs = ref([
      {
        id: 1,
        name: '夜空中最亮的星',
        artist: '逃跑计划',
        duration: 280,
        genre: '摇滚',
        playCount: 22,
        isNew: false,
        cover: 'https://images.unsplash.com/photo-1514525253161-7a46d19cd819?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/夜空中最亮的星.mp3'
      },
      {
        id: 2,
        name: '夏天的风',
        artist: '温岚',
        duration: 210,
        genre: '流行',
        playCount: 8,
        isNew: false,
        cover: 'https://images.unsplash.com/photo-1508700115892-45ecd05ae2ad?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/夏天的风-温岚.mp3'
      },
      {
        id: 3,
        name: '平凡之路',
        artist: '朴树',
        duration: 316,
        genre: '摇滚',
        playCount: 25,
        isNew: false,
        cover: 'https://images.unsplash.com/photo-1511671782779-c97d3d27a1d4?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/平凡之路-朴树-标准.mp3'
      },
      {
        id: 4,
        name: '光年之外',
        artist: 'G.E.M.邓紫棋',
        duration: 235,
        genre: '流行',
        playCount: 18,
        isNew: false,
        cover: 'https://images.unsplash.com/photo-1518609878373-06d740f60d8b?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/光年之外-邓紫棋-标准.mp3'
      },
      {
        id: 5,
        name: '起风了',
        artist: '买辣椒也用券',
        duration: 325,
        genre: '流行',
        playCount: 15,
        isNew: false,
        cover: 'https://images.unsplash.com/photo-1470225620780-dba8ba36b745?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/起风了-买辣椒也用券.mp3'
      },
      {
        id: 6,
        name: '年轮',
        artist: '张碧晨',
        duration: 245,
        genre: '流行',
        playCount: 12,
        isNew: false,
        cover: 'https://images.unsplash.com/photo-1493225457124-a3eb161ffa5f?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/年轮-张碧晨-标准.mp3'
      },
      {
        id: 7,
        name: '卡农',
        artist: '古典名曲',
        duration: 320,
        genre: '古典',
        playCount: 5,
        isNew: true,
        cover: 'https://images.unsplash.com/photo-1511379938547-c1f69419868d?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/卡农-纯音乐-标准.mp3'
      },
      {
        id: 8,
        name: '月光奏鸣曲',
        artist: '贝多芬',
        duration: 360,
        genre: '古典',
        playCount: 3,
        isNew: true,
        cover: 'https://images.unsplash.com/photo-1511735111819-9a3f7709049c?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/月光奏鸣曲-贝多芬.mp3'
      },
      {
        id: 9,
        name: '海阔天空',
        artist: 'Beyond',
        duration: 310,
        genre: '摇滚',
        playCount: 20,
        isNew: false,
        cover: 'https://images.unsplash.com/photo-1516450360452-9312f5e86fc7?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/海阔天空-Beyond-标准.mp3'
      },
      {
        id: 10,
        name: '小幸运',
        artist: '田馥甄',
        duration: 275,
        genre: '流行',
        playCount: 10,
        isNew: false,
        cover: 'https://images.unsplash.com/photo-1516280440614-37939bbacd81?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/小幸运-田馥甄-标准.mp3'
      },
      {
        id: 11,
        name: '富士山下',
        artist: '陈奕迅',
        duration: 240,
        genre: '经典',
        playCount: 14,
        isNew: false,
        cover: 'https://images.unsplash.com/photo-1459749411175-04bf5292ceea?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/富士山下-陈奕迅-高品.mp3'
      },
      {
        id: 12,
        name: 'River Flows In You',
        artist: 'Yiruma',
        duration: 220,
        genre: '纯音乐',
        playCount: 6,
        isNew: true,
        cover: 'https://images.unsplash.com/photo-1511379938547-c1f69419868d?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80',
        audioUrl: 'audio/River Flows in You.mp3'
      }
    ])
    
    // 歌单数据（6个完整歌单）
    const playlists = ref([
      {
        id: 1,
        name: '治愈系歌单',
        description: '温暖心灵的旋律，带你走出阴霾，感受阳光',
        songCount: 6,
        songs: [1, 2, 4, 5, 10],
        cover: 'https://images.unsplash.com/photo-1516280440614-37939bbacd81?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80'
      },
      {
        id: 2,
        name: '复古粤语金曲',
        description: '经典粤语歌曲回顾，重温黄金年代的旋律',
        songCount: 3,
        songs: [9, 11],
        cover: 'https://images.unsplash.com/photo-1470225620780-dba8ba36b745?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80'
      },
      {
        id: 3,
        name: '纯音乐',
        description: '无歌词的纯粹音乐享受，适合工作学习时聆听',
        songCount: 4,
        songs: [7, 8, 12],
        cover: 'https://images.unsplash.com/photo-1511735111819-9a3f7709049c?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80'
      },
      {
        id: 4,
        name: '伤感',
        description: '触动心弦的情感旋律，适合安静时细细品味',
        songCount: 4,
        songs: [1, 5, 6, 10],
        cover: 'https://images.unsplash.com/photo-1459749411175-04bf5292ceea?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80'
      },
      {
        id: 5,
        name: '摇滚力量',
        description: '充满力量的摇滚音乐，释放内心的激情',
        songCount: 4,
        songs: [3, 6, 9],
        cover: 'https://images.unsplash.com/photo-1511671782779-c97d3d27a1d4?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80'
      },
      {
        id: 6,
        name: '工作学习',
        description: '专注时听的背景音乐，提高工作效率',
        songCount: 5,
        songs: [7, 8, 12],
        cover: 'https://images.unsplash.com/photo-1511379938547-c1f69419868d?ixlib=rb-4.0.3&auto=format&fit=crop&w=600&q=80'
      }
    ])
    
    // 当前歌单的歌曲
    const playlistSongs = computed(() => {
      if (!currentPlaylist.value) return []
      return songs.value.filter(song => 
        currentPlaylist.value.songs.includes(song.id)
      )
    })
    
    // 所有歌曲风格（包含所有6种风格）
    const genres = computed(() => {
      const allGenres = songs.value.map(song => song.genre)
      const uniqueGenres = ['all', ...new Set(allGenres)]
      return uniqueGenres
    })
    
    // 根据风格筛选的歌曲
    const filteredByGenre = computed(() => {
      if (currentGenre.value === 'all') {
        return songs.value
      }
      return songs.value.filter(song => song.genre === currentGenre.value)
    })
    
    // 根据搜索词筛选的歌曲
    const filteredSongs = computed(() => {
      let result = filteredByGenre.value
      
      if (searchQuery.value.trim() !== '') {
        const query = searchQuery.value.toLowerCase().trim()
        result = result.filter(song => 
          song.name.toLowerCase().includes(query) || 
          song.artist.toLowerCase().includes(query)
        )
      }
      
      return result
    })
    
    // 收藏的歌曲
    const favoriteSongs = computed(() => {
      return songs.value.filter(song => favorites.value.includes(song.id))
    })
    
    // 播放次数最多的歌曲
    const mostPlayedSongs = computed(() => {
      return [...songs.value]
        .sort((a, b) => b.playCount - a.playCount)
        .slice(0, 5)
    })
    
    // 推荐新歌
    const unplayedSongs = computed(() => {
      return songs.value
        .filter(song => song.playCount < 5 || song.isNew)
        .sort((a, b) => b.isNew - a.isNew || a.playCount - b.playCount)
        .slice(0, 6)
    })
    
    // 播放/暂停切换 - 简化版本
    const togglePlay = () => {
      if (!currentSong.value) {
        playLastPlayedOrRandom()
        return
      }
      
      if (!audioElement.value) return
      
      if (isPlaying.value) {
        audioElement.value.pause()
        isPlaying.value = false
      } else {
        // 如果音频没有加载，重新加载
        if (!audioElement.value.src || audioElement.value.src !== currentSong.value.audioUrl) {
          audioElement.value.src = currentSong.value.audioUrl
          audioElement.value.load()
        }
        
        const playPromise = audioElement.value.play()
        
        if (playPromise !== undefined) {
          playPromise
            .then(() => {
              isPlaying.value = true
              isLoading.value = false
            })
            .catch(error => {
              console.error('播放失败:', error)
              isPlaying.value = false
              showNotification('播放失败，请检查音频文件')
            })
        }
      }
    }
    
    // 播放指定歌曲
    const playSong = (song) => {
      if (currentSong.value && currentSong.value.id === song.id && isPlaying.value) {
        togglePlay()
        return
      }
      
      if (!audioElement.value) return
      
      // 设置加载状态
      isLoading.value = true
      isPlaying.value = false
      
      // 暂停当前音频
      audioElement.value.pause()
      
      // 设置新歌曲
      currentSong.value = song
      
      // 更新音频源
      audioElement.value.src = song.audioUrl
      audioElement.value.volume = volume.value
      
      // 加载音频
      audioElement.value.load()
      
      // 尝试播放
      const playPromise = audioElement.value.play()
      
      if (playPromise !== undefined) {
        playPromise
          .then(() => {
            isPlaying.value = true
            isLoading.value = false
            
            // 更新播放次数
            updatePlayCount(song.id)
            
            // 保存为最后播放的歌曲
            lastPlayedSong.value = song
            
            // 添加到最近播放列表
            addToRecentlyPlayed(song)
            
            // 在随机模式下记录已播放的歌曲
            if (playMode.value === 2) {
              playedInRandomMode.value.push(song.id)
              if (playedInRandomMode.value.length >= songs.value.length) {
                playedInRandomMode.value = [song.id]
              }
            }
          })
          .catch(error => {
            console.error('播放失败:', error)
            isPlaying.value = false
            isLoading.value = false
            
            // 设置超时，稍后自动重试一次
            setTimeout(() => {
              if (!isPlaying.value && currentSong.value && currentSong.value.id === song.id) {
                const retryPromise = audioElement.value.play()
                if (retryPromise !== undefined) {
                  retryPromise.catch(() => {
                    showNotification('音频加载失败，请检查网络连接')
                  })
                }
              }
            }, 500)
          })
      }
    }
    
    // 更新播放次数
    const updatePlayCount = (songId) => {
      const songIndex = songs.value.findIndex(song => song.id === songId)
      if (songIndex !== -1) {
        songs.value[songIndex].playCount += 1
        localStorage.setItem('playCounts', JSON.stringify(
          songs.value.reduce((acc, song) => {
            acc[song.id] = song.playCount
            return acc
          }, {})
        ))
      }
    }
    
    // 播放上次播放的歌曲或随机一首
    const playLastPlayedOrRandom = () => {
      if (lastPlayedSong.value) {
        playSong(lastPlayedSong.value)
      } else if (recentlyPlayed.value.length > 0) {
        playSong(recentlyPlayed.value[0])
      } else {
        const randomSong = songs.value[Math.floor(Math.random() * songs.value.length)]
        playSong(randomSong)
      }
    }
    
    // 播放随机热门歌曲
    const playRandomHotSong = () => {
      if (mostPlayedSongs.value.length > 0) {
        const randomIndex = Math.floor(Math.random() * Math.min(3, mostPlayedSongs.value.length))
        playSong(mostPlayedSongs.value[randomIndex])
      } else {
        const randomSong = songs.value[Math.floor(Math.random() * songs.value.length)]
        playSong(randomSong)
      }
    }
    
    // 播放推荐新歌
    const playRecommendedNewSong = () => {
      if (unplayedSongs.value.length > 0) {
        const randomIndex = Math.floor(Math.random() * Math.min(3, unplayedSongs.value.length))
        playSong(unplayedSongs.value[randomIndex])
      } else {
        const leastPlayed = [...songs.value].sort((a, b) => a.playCount - b.playCount)[0]
        playSong(leastPlayed)
      }
    }
    
    // 播放歌单的所有歌曲
    const playAllSongs = () => {
      if (playlistSongs.value.length > 0) {
        playSong(playlistSongs.value[0])
      }
    }
    
    // 查看歌单
    const viewPlaylist = (playlist) => {
      currentPlaylist.value = playlist
      currentPage.value = 'playlist'
    }
    
    // 添加到最近播放列表
    const addToRecentlyPlayed = (song) => {
      const index = recentlyPlayed.value.findIndex(item => item.id === song.id)
      if (index !== -1) {
        recentlyPlayed.value.splice(index, 1)
      }
      
      recentlyPlayed.value.unshift(song)
      
      if (recentlyPlayed.value.length > 10) {
        recentlyPlayed.value = recentlyPlayed.value.slice(0, 10)
      }
      
      localStorage.setItem('recentlyPlayed', JSON.stringify(recentlyPlayed.value))
    }
    
    // 获取下一首歌曲
    const getNextSong = () => {
      if (!currentSong.value || songs.value.length === 0) return null
      
      const currentIndex = songs.value.findIndex(song => song.id === currentSong.value.id)
      
      if (playMode.value === 0) {
        // 顺序播放模式
        const nextIndex = (currentIndex + 1) % songs.value.length
        return songs.value[nextIndex]
      } else if (playMode.value === 2) {
        // 随机播放模式
        let availableSongs = songs.value
        
        if (songs.value.length > 1) {
          availableSongs = songs.value.filter(song => song.id !== currentSong.value.id)
        }
        
        if (availableSongs.length > 1 && playedInRandomMode.value.length > 0) {
          const recentlyPlayedIds = playedInRandomMode.value.slice(-3)
          availableSongs = availableSongs.filter(song => !recentlyPlayedIds.includes(song.id))
        }
        
        if (availableSongs.length === 0) {
          availableSongs = songs.value.filter(song => song.id !== currentSong.value.id)
        }
        
        const randomIndex = Math.floor(Math.random() * availableSongs.length)
        return availableSongs[randomIndex]
      }
      
      return null
    }
    
    // 获取上一首歌曲
    const getPreviousSong = () => {
      if (!currentSong.value || songs.value.length === 0) return null
      
      const currentIndex = songs.value.findIndex(song => song.id === currentSong.value.id)
      
      if (playMode.value === 0) {
        // 顺序播放模式
        const prevIndex = (currentIndex - 1 + songs.value.length) % songs.value.length
        return songs.value[prevIndex]
      } else if (playMode.value === 2) {
        // 随机播放模式
        return getNextSong()
      }
      
      return null
    }
    
    // 下一首
    const nextSong = () => {
      const next = getNextSong()
      if (next) {
        playSong(next)
      }
    }
    
    // 上一首
    const previousSong = () => {
      const prev = getPreviousSong()
      if (prev) {
        playSong(prev)
      }
    }
    
    // 切换播放模式
    const changePlayMode = () => {
      playMode.value = (playMode.value + 1) % 3
      
      if (playMode.value === 2) {
        playedInRandomMode.value = []
        if (currentSong.value) {
          playedInRandomMode.value.push(currentSong.value.id)
        }
      }
      
      localStorage.setItem('playMode', playMode.value.toString())
    }
    
    // 切换收藏状态
    const toggleFavorite = (songId) => {
      const index = favorites.value.indexOf(songId)
      if (index === -1) {
        favorites.value.push(songId)
      } else {
        favorites.value.splice(index, 1)
      }
      
      localStorage.setItem('musicFavorites', JSON.stringify(favorites.value))
    }
    
    // 按风格筛选歌曲
    const filterByGenre = (genre) => {
      currentGenre.value = genre
    }
    
    // 更新搜索查询
    const updateSearch = (query) => {
      searchQuery.value = query
    }
    
    // 清除搜索
    const clearSearch = () => {
      searchQuery.value = ''
    }
    
    // 调整播放进度
    const seekTo = (event) => {
      if (!audioElement.value || !currentSong.value) return
      
      const progressBar = event.currentTarget
      const clickPosition = event.offsetX
      const progressBarWidth = progressBar.clientWidth
      const percent = clickPosition / progressBarWidth
      
      const newTime = percent * duration.value
      audioElement.value.currentTime = newTime
      currentTime.value = newTime
    }
    
    // 调整音量
    const adjustVolume = (event) => {
      if (!audioElement.value) return
      
      const volumeBar = event.currentTarget
      const clickPosition = event.offsetX
      const volumeBarWidth = volumeBar.clientWidth
      volume.value = Math.max(0, Math.min(1, clickPosition / volumeBarWidth))
      
      audioElement.value.volume = volume.value
      localStorage.setItem('volume', volume.value.toString())
    }
    
    // 音频事件处理
    const handleTimeUpdate = () => {
      if (audioElement.value) {
        currentTime.value = audioElement.value.currentTime
      }
    }
    
    const handleLoadedMetadata = () => {
      if (audioElement.value) {
        duration.value = audioElement.value.duration
        
        // 如果歌曲数据中的duration不准确，更新它
        if (currentSong.value) {
          currentSong.value.duration = duration.value
        }
      }
    }
    
    const handleSongEnded = () => {
      if (playMode.value === 1) {
        // 单曲循环模式：重新播放当前歌曲
        audioElement.value.currentTime = 0
        const playPromise = audioElement.value.play()
        if (playPromise !== undefined) {
          playPromise.catch(error => {
            console.error('循环播放失败:', error)
            isPlaying.value = false
          })
        }
      } else {
        // 顺序播放或随机播放：播放下一首
        nextSong()
      }
    }
    
    const handleAudioError = (error) => {
      console.error('音频错误:', error)
      isLoading.value = false
      isPlaying.value = false
      
      showNotification('音频加载失败，请检查网络连接或更换音频文件')
    }
    
    // 格式化时间
    const formatTime = (seconds) => {
      if (!seconds) return '0:00'
      const mins = Math.floor(seconds / 60)
      const secs = Math.floor(seconds % 60)
      return `${mins}:${secs.toString().padStart(2, '0')}`
    }
    
    // 计算播放进度百分比
    const progressPercent = computed(() => {
      if (!duration.value || duration.value === 0) return 0
      return (currentTime.value / duration.value) * 100
    })
    
    // 计算总时长
    const totalDuration = computed(() => {
      const totalSeconds = songs.value.reduce((sum, song) => sum + song.duration, 0)
      return Math.round(totalSeconds / 60)
    })
    
    // 格式化歌曲时长
    const formatDuration = (seconds) => {
      const mins = Math.floor(seconds / 60)
      const secs = Math.floor(seconds % 60)
      return `${mins}:${secs.toString().padStart(2, '0')}`
    }
    
    // 切换页面
    const changePage = (page) => {
      currentPage.value = page
      currentPlaylist.value = null
    }
    
    // 返回上一页
    const goBack = () => {
      if (currentPage.value === 'playlist') {
        currentPage.value = 'home'
        currentPlaylist.value = null
      }
    }
    
    // 显示通知
    const showNotification = (message) => {
      // 在实际项目中，可以使用更优雅的通知组件
      alert(message)
    }
    
    // 监听搜索查询变化
    watch(searchQuery, (newValue) => {
      localStorage.setItem('searchQuery', newValue)
    })
    
    // 初始化
    onMounted(() => {
      // 加载收藏列表
      const savedFavorites = localStorage.getItem('musicFavorites')
      if (savedFavorites) {
        favorites.value = JSON.parse(savedFavorites)
      }
      
      // 加载最近播放列表
      const savedRecentlyPlayed = localStorage.getItem('recentlyPlayed')
      if (savedRecentlyPlayed) {
        recentlyPlayed.value = JSON.parse(savedRecentlyPlayed)
      }
      
      // 加载播放次数
      const savedPlayCounts = localStorage.getItem('playCounts')
      if (savedPlayCounts) {
        const counts = JSON.parse(savedPlayCounts)
        songs.value.forEach(song => {
          if (counts[song.id]) {
            song.playCount = counts[song.id]
          }
        })
      }
      
      // 加载播放模式
      const savedPlayMode = localStorage.getItem('playMode')
      if (savedPlayMode !== null) {
        playMode.value = parseInt(savedPlayMode)
      }
      
      // 加载音量设置
      const savedVolume = localStorage.getItem('volume')
      if (savedVolume !== null) {
        volume.value = parseFloat(savedVolume)
      }
      
      // 加载搜索查询
      const savedSearchQuery = localStorage.getItem('searchQuery')
      if (savedSearchQuery !== null) {
        searchQuery.value = savedSearchQuery
      }
      
      // 设置最后播放的歌曲
      if (recentlyPlayed.value.length > 0) {
        lastPlayedSong.value = recentlyPlayed.value[0]
      }
      
      // 初始化音频元素
      if (audioElement.value) {
        audioElement.value.volume = volume.value
      }
      
      // 默认播放第一首歌（不自动播放）
      if (songs.value.length > 0 && !currentSong.value) {
        currentSong.value = songs.value[0]
      }
      
      // 初始化随机播放记录
      if (playMode.value === 2 && currentSong.value) {
        playedInRandomMode.value = [currentSong.value.id]
      }
    })
    
    return {
      currentPage,
      songs,
      playlists,
      currentSong,
      isPlaying,
      currentTime,
      duration,
      volume,
      playMode,
      playModeText,
      playModeIcon,
      favorites,
      recentlyPlayed,
      lastPlayedSong,
      mostPlayedSongs,
      unplayedSongs,
      currentGenre,
      genres,
      filteredSongs,
      favoriteSongs,
      currentPlaylist,
      playlistSongs,
      searchQuery,
      progressPercent,
      totalDuration,
      isLoading,
      audioElement,
      togglePlay,
      playSong,
      playLastPlayedOrRandom,
      playRandomHotSong,
      playRecommendedNewSong,
      playAllSongs,
      viewPlaylist,
      nextSong,
      previousSong,
      changePlayMode,
      toggleFavorite,
      filterByGenre,
      updateSearch,
      clearSearch,
      seekTo,
      adjustVolume,
      formatTime,
      formatDuration,
      changePage,
      goBack,
      handleTimeUpdate,
      handleLoadedMetadata,
      handleSongEnded,
      handleAudioError
    }
  }
}
</script>

<style>
/* 基础样式
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: #fff;
  min-height: 100vh;
  padding-bottom: 100px;
}

#app {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

main {
  margin-top: 20px;
} */

/* 响应式设计 */
@media (max-width: 768px) {
  #app {
    padding: 10px;
  }
}

/* 隐藏音频元素 */
audio {
  display: none;
}

/* 加载动画 */
@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.loading-spinner {
  display: inline-block;
  width: 20px;
  height: 20px;
  border: 3px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top-color: #fff;
  animation: spin 1s ease-in-out infinite;
}
</style>