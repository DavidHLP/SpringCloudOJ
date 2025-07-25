<template>
  <div class="left-sidebar">

    <BigTagesFilter></BigTagesFilter>
    <!-- 题目表格区域 -->
    <div class="content-section">
      <QuestionSearch></QuestionSearch>
      <QuestionTable ref="questionTableRef" :questions="questions" :loading="loading" :has-more="hasMore"
        @load-more="loadMoreQuestions" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { ElMessage } from 'element-plus'
import QuestionTable from './QuestionTable.vue'
import BigTagesFilter from './BigTagesFilter.vue'
import { getQuestionBank } from '@/api/questionbank'
import type { Question, QuestionBankQuery } from '@/types/questionbank'
import QuestionSearch from './QuestionSearch.vue'

// 定义事件
const emit = defineEmits<{
  (e: 'questions-loaded', count: number): void
}>()

// 组件引用
const questionTableRef = ref<InstanceType<typeof QuestionTable>>()

// 响应式数据
const loading = ref(false)
const questions = ref<Question[]>([])
const currentPage = ref(1)
const pageSize = ref(20)
const hasMore = ref(true)

// 获取题目数据
const fetchQuestions = async (isLoadMore = false, params: Partial<QuestionBankQuery> = {}) => {
  if (loading.value) return;
  loading.value = true;

  try {
    const queryParams: QuestionBankQuery = {
      page: currentPage.value,
      size: pageSize.value,
      ...params
    };

    const res = await getQuestionBank(queryParams);
    const newQuestions = res.records;

    if (isLoadMore) {
      questions.value.push(...newQuestions);
    } else {
      questions.value = newQuestions;
    }

    hasMore.value = questions.value.length < res.total;
    emit('questions-loaded', questions.value.length);
  } catch (error) {
    ElMessage.error('获取题目列表失败');
    console.error('Error fetching questions:', error);
  } finally {
    loading.value = false;
  }
};

// 处理搜索
const handleSearch = (query: string) => {
  currentPage.value = 1
  fetchQuestions(false, { searchQuery: query })
}

// 处理排序
const handleSortChange = (sortBy: string) => {
  currentPage.value = 1
  fetchQuestions(false, { sortBy })
}

// 加载更多题目
const loadMoreQuestions = () => {
  if (!hasMore.value) return
  currentPage.value++
  fetchQuestions(true)
}

// 暴露方法给父组件
defineExpose({
  loadMoreQuestions,
  handleSearch,
  handleSortChange
})

// 生命周期
onMounted(() => {
  fetchQuestions()
})
</script>

<style scoped>
.left-sidebar {
  display: flex;
  flex-direction: column;
  gap: 24px;
  width: 100%;
}

/* 页面标题 */
.page-header {
  margin-bottom: 8px;
}

.page-title {
  font-size: 1.5rem;
  line-height: 2rem;
  font-weight: 600;
  color: #1f2937;
  margin: 0 0 0.5rem 0;
}

.page-description {
  color: #4b5563;
  margin: 0;
  font-size: 0.875rem;
}

/* 筛选区域 */
.filter-section {
  background: #fff;
  border-radius: 12px;
  padding: 20px;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
  border: 1px solid #f0f0f0;
}

.filter-controls {
  display: flex;
  gap: 16px;
  align-items: center;
  flex-wrap: wrap;
}

.filter-item {
  min-width: 120px;
}

.search-item {
  flex: 1;
  min-width: 200px;
}

.filter-select {
  width: 100%;
}

.search-input {
  width: 100%;
}

/* Element Plus 组件样式优化 */
:deep(.el-input__wrapper) {
  border-radius: 8px;
  box-shadow: 0 1px 3px rgba(0, 0, 0, 0.06);
  border: 1px solid #e0e0e0;
  transition: all 0.3s;
}

:deep(.el-input__wrapper.is-focus) {
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2);
  border-color: #409eff;
}

:deep(.el-select .el-input__wrapper) {
  border-radius: 8px;
}

:deep(.el-select .el-input__wrapper.is-focus) {
  box-shadow: 0 0 0 2px rgba(64, 158, 255, 0.2);
  border-color: #409eff;
}

.search-btn {
  padding: 0 8px;
  margin-right: -8px;
  color: #909399;
  transition: color 0.2s;
}

.search-btn:hover {
  color: #409eff;
}

/* 内容区域 */
.content-section {
  flex: 1;
}

.table-card {
  border-radius: 12px;
  border: 1px solid #f0f0f0;
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.04);
}

:deep(.el-card__body) {
  padding: 20px;
}

/* 响应式调整 */
@media (max-width: 1024px) {
  .filter-controls {
    flex-direction: column;
    gap: 12px;
  }

  .filter-item,
  .search-item {
    width: 100%;
    min-width: auto;
  }
}

@media (max-width: 768px) {
  .left-sidebar {
    gap: 16px;
  }

  .filter-section {
    padding: 16px;
  }

  .page-title {
    font-size: 1.25rem;
    line-height: 1.75rem;
  }
}
</style>
