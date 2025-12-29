<template>
  <div class="filter-bar">
    <h4><i class="fas fa-filter"></i> 歌曲筛选</h4>
    
    <div class="filter-group">
      <div class="search-box">
        <i class="fas fa-search"></i>
        <input 
          type="text" 
          placeholder="搜索歌曲或歌手..." 
          :value="searchQuery"
          @input="$emit('update-search', $event.target.value)"
          class="search-input"
        >
        <button v-if="searchQuery" @click="$emit('clear-search')" class="clear-search">
          <i class="fas fa-times"></i>
        </button>
      </div>
    </div>
    
    <div class="filter-options">
      <button 
        v-for="genre in genres" 
        :key="genre"
        class="filter-btn"
        :class="{ active: currentGenre === genre }"
        @click="$emit('filter-by-genre', genre)"
      >
        {{ genre === 'all' ? '全部' : genre }}
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'FilterBar',
  props: {
    genres: {
      type: Array,
      required: true
    },
    currentGenre: {
      type: String,
      required: true
    },
    searchQuery: {
      type: String,
      default: ''
    }
  },
  emits: ['filter-by-genre', 'update-search', 'clear-search']
}
</script>

<style scoped>
.filter-bar {
  background: rgba(255, 255, 255, 0.08);
  border-radius: 15px;
  padding: 20px;
  margin-bottom: 25px;
}

.filter-bar h4 {
  font-size: 1.2rem;
  margin-bottom: 15px;
  color: #fff;
  display: flex;
  align-items: center;
  gap: 10px;
}

.filter-group {
  margin-bottom: 15px;
}

.search-box {
  display: flex;
  align-items: center;
  background: rgba(255, 255, 255, 0.1);
  border-radius: 25px;
  padding: 10px 15px;
  gap: 10px;
}

.search-box i {
  color: rgba(255, 255, 255, 0.7);
}

.search-input {
  flex: 1;
  background: transparent;
  border: none;
  color: #fff;
  font-size: 0.9rem;
  outline: none;
  width: 100%;
}

.search-input::placeholder {
  color: rgba(255, 255, 255, 0.5);
}

.clear-search {
  background: none;
  border: none;
  color: rgba(255, 255, 255, 0.7);
  cursor: pointer;
  width: 24px;
  height: 24px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: all 0.3s;
}

.clear-search:hover {
  background: rgba(255, 255, 255, 0.1);
  color: #fff;
}

.filter-options {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.filter-btn {
  background: rgba(255, 255, 255, 0.1);
  border: none;
  color: #fff;
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-size: 0.9rem;
  transition: all 0.3s;
}

.filter-btn:hover {
  background: rgba(255, 255, 255, 0.2);
}

.filter-btn.active {
  background: #4cc9f0;
  color: #16213e;
  font-weight: 500;
}
</style>