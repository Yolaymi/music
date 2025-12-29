<template>
  <div class="playlist-card" @click="$emit('view-playlist', playlist)">
    <div class="playlist-cover">
      <img :src="playlist.cover" :alt="playlist.name">
    </div>
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