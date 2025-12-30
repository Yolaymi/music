<template>
  <!-- 筛选栏容器 -->
  <div class="filter-bar">
    <h4><i class="fas fa-filter"></i> 歌曲筛选</h4>
    <!-- 筛选组：包含搜索框 -->
    <div class="filter-group">
      <div class="search-box">
        <i class="fas fa-search"></i>
        <!-- 绑定父组件传递的搜索关键词（回显搜索内容）、输入内容变化时触发事件 -->
        <input 
          type="text" 
          placeholder="搜索歌曲或歌手..." 
          :value="searchQuery"
          @input="$emit('update-search', $event.target.value)"
          class="search-input"
        >
        <!-- 清空搜索按钮：当搜索关键词非空时显示 -->
        <button v-if="searchQuery" @click="$emit('clear-search')" class="clear-search">
          <i class="fas fa-times"></i>
        </button>
      </div>
    </div>
    <!-- 筛选组：包含风格筛选按钮 -->
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
/* 空状态样式：无搜索结果时显示 */
.empty-state {
  text-align: center; /* 文本居中 */
  padding: 60px 20px; /* 上下内边距60px，左右20px */
  color: rgba(255, 255, 255, 0.6); /* 浅灰色文字 */
}

/* 空状态图标样式：大尺寸、主题蓝色 */
.empty-state i {
  font-size: 4rem; /* 图标大小 */
  margin-bottom: 20px; /* 底部间距 */
  color: #4cc9f0; /* 主题蓝色 */
}

/* 空状态标题样式：稍大字体、较浅白色 */
.empty-state h3 {
  font-size: 1.8rem;
  margin-bottom: 10px;
  color: rgba(255, 255, 255, 0.8);
}

/* 空状态描述文本样式：适中字体、换行限制宽度、行高优化 */
.empty-state p {
  font-size: 1.1rem;
  max-width: 400px; /* 最大宽度，避免文字过长 */
  margin: 0 auto; /* 水平居中 */
  line-height: 1.6; /* 行高，提升可读性 */
}

/* 音乐库统计卡片容器：弹性布局、间距、自动换行 */
.library-stats {
  display: flex;
  gap: 20px; /* 卡片之间的间距 */
  margin-bottom: 30px; /* 底部间距，与筛选栏分隔 */
  flex-wrap: wrap; /* 屏幕变小时自动换行 */
}

/* 统计卡片样式：半透明背景、圆角、内边距、弹性布局 */
.stat-card {
  background: rgba(255, 255, 255, 0.08); /* 半透明白色背景（适配深色主题） */
  padding: 20px; /* 内边距 */
  border-radius: 15px; /* 圆角 */
  flex: 1; /* 卡片均匀分配剩余空间 */
  min-width: 200px; /* 最小宽度，避免卡片过窄 */
  display: flex;
  align-items: center; /* 图标和文本垂直居中 */
  gap: 20px; /* 图标和文本间距 */
}

/* 统计卡片图标样式：大尺寸、主题蓝色 */
.stat-card i {
  font-size: 2.5rem;
  color: #4cc9f0;
}

/* 统计卡片小标题样式：小字体、浅灰色 */
.stat-card h4 {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  margin-bottom: 5px; /* 与数值间距 */
}

/* 统计卡片数值样式：大字体、加粗、白色 */
.stat-card p {
  font-size: 1.5rem;
  font-weight: bold;
  color: #fff;
}

/* 响应式样式：屏幕宽度≤768px时（手机端） */
@media (max-width: 768px) {
  .library-stats {
    flex-direction: column; /* 统计卡片垂直排列 */
  }
  
  .stat-card {
    width: 100%; /* 卡片占满屏幕宽度 */
  }
}
</style>