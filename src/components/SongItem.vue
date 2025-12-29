<template>
  <div 
    class="song-item"
    :class="{ playing: isCurrent && isPlaying }"
  >
    <div class="album-cover">
      <img :src="song.cover" :alt="song.name">
    </div>
    <div class="song-details" @click="$emit('play-song', song)">
      <h3>{{ song.name }}</h3>
      <p>{{ song.artist }} • {{ formatDuration(song.duration) }}</p>
    </div>
    <div class="song-actions">
      <button @click="$emit('play-song', song)" class="play-btn-small">
        <i class="fas" :class="isCurrent && isPlaying ? 'fa-pause' : 'fa-play'"></i>
      </button>
      <button @click="$emit('toggle-favorite', song.id)" class="favorite-btn">
        <i class="fas" :class="isFavorite ? 'fa-heart' : 'fa-heart-o'"></i>
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'SongItem',
  props: {
    song: {
      type: Object,
      required: true
    },
    isCurrent: {
      type: Boolean,
      default: false
    },
    isPlaying: {
      type: Boolean,
      default: false
    },
    isFavorite: {
      type: Boolean,
      default: false
    }
  },
  emits: ['play-song', 'toggle-favorite'],
  setup() {
    const formatDuration = (seconds) => {
      const mins = Math.floor(seconds / 60)
      const secs = seconds % 60
      return `${mins}:${secs.toString().padStart(2, '0')}`
    }
    
    return {
      formatDuration
    }
  }
}
</script>