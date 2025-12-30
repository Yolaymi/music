<template>
  <div class="page">
    <div class="my-page">
      <h2><i class="fas fa-user"></i> 我的页面</h2>
      <p class="page-description">管理你的收藏和查看播放历史</p>
      <!-- 我的页面核心内容区：包含“我的收藏”和“最近播放”两个模块 -->
      <div class="my-page-sections">
        <!-- 模块1：我的收藏 -->
        <div class="my-page-section">
          <h3><i class="fas fa-heart"></i> 我的收藏</h3>

          <!-- 收藏空状态：没有收藏歌曲时显示 -->
          <div v-if="favoriteSongs.length === 0" class="empty-state" style="padding: 30px 20px;">
            <i class="fas fa-heart-broken"></i>
            <h3>暂无收藏的歌曲</h3>
            <p>去音乐库发现你喜欢的音乐并收藏它们吧！</p>
          </div>

          <!-- 收藏歌曲列表：有收藏歌曲时显示 -->
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
              <!-- 监听播放事件，传发给父组件播放、监听取消收藏事件，转发给父组件 -->
            </div>
          </div>
        </div>
        
        <!-- 模块2：最近播放 -->
        <div class="my-page-section">
          <h3><i class="fas fa-history"></i> 最近播放</h3>
          
          <!-- 最近播放空状态：没有播放记录时显示 -->
          <div v-if="recentlyPlayed.length === 0" class="empty-state" style="padding: 30px 20px;">
            <i class="fas fa-clock"></i>
            <h3>暂无播放记录</h3>
            <p>开始播放音乐，这里会显示你的播放历史</p>
          </div>
          
          <!-- 最近播放列表：有播放记录时显示 -->
          <div v-else>
            <div class="song-list">
              <!-- 直接渲染最近播放歌曲项 -->
              <div 
                v-for="song in recentlyPlayed" 
                :key="song.id"
                class="song-item"
                :class="{ playing: currentSong && currentSong.id === song.id }"
                @click="$emit('play-song', song)"
              >
                <!-- 专辑封面 -->
                <div class="album-cover">
                  <img :src="song.cover" :alt="song.name">
                </div>
                <!-- 歌曲详情（名称、歌手） -->
                <div class="song-details">
                  <h3>{{ song.name }}</h3>
                  <p>{{ song.artist }}</p>
                </div>
                 <!-- 操作按钮区（播放/收藏） -->
                <div class="song-actions">
                  <!-- 播放/暂停按钮：点击触发播放事件 -->
                  <button @click="$emit('play-song', song)" class="play-btn-small">
                    <i class="fas" :class="currentSong && currentSong.id === song.id && isPlaying ? 'fa-pause' : 'fa-play'"></i>
                  </button>
                  <!-- 收藏/取消收藏按钮：点击触发收藏切换事件 -->
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
  // 接收父组件传递的props数据（定义类型、必填项、默认值）
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
  // 声明当前组件要触发的自定义事件（向父组件传递交互消息）
  emits: ['play-song', 'toggle-favorite']
}
</script>