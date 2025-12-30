<template>
  <div class="playlist-card" @click="$emit('view-playlist', playlist)">
    <!-- 歌单封面区域 -->
    <div class="playlist-cover">
      <img :src="playlist.cover" :alt="playlist.name">
    </div>
    <!-- 歌单信息区域（名称+歌曲数量） -->
    <div class="playlist-info">
      <h4>{{ playlist.name }}</h4>
      <p>{{ playlist.songCount }} 首歌曲</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'PlaylistCard',
  props: {
    playlist: {
      type: Object,
      required: true
    }
  },
  emits: ['view-playlist']
}
</script>