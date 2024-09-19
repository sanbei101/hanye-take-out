<template>
  <div class="container">
    <div class="homeTitle">
      <h2>
        今日数据<i>{{ days[1] }}</i>
      </h2>
      <div class="more">
        <router-link to="statistics">详细数据</router-link>
        <el-icon>
          <ArrowRight />
        </el-icon>
      </div>
    </div>
    <div class="overviewBox">
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          营业额
        </template>
        <div class="card-content">¥ {{ props.overviewData.turnover }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          有效订单
        </template>
        <div class="card-content">{{ props.overviewData.validOrderCount }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          订单完成率
        </template>
        <div class="card-content">{{ (props.overviewData.orderCompletionRate * 100).toFixed(0) }}%</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          平均客单价
        </template>
        <div class="card-content">¥ {{ props.overviewData.unitPrice }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          新增用户
        </template>
        <div class="card-content">{{ props.overviewData.newUsers }}</div>
      </el-card>
    </div>
  </div>
</template>

<style scoped>
.homeTitle {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.more {
  display: flex;
  align-items: center;
}

.overviewBox {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
  padding: 0.5rem 1rem;
}

.overviewCard {
  flex: 1;
  min-width: 200px;
}


.card-content {
  font-size: 24px;
  font-weight: bold;
}

@media (max-width: 768px) {
  .overviewCard {
    flex-basis: calc(50% - 8px);
  }
}

@media (max-width: 480px) {
  .overviewCard {
    flex-basis: 100%;
  }
}
</style>

<script lang="ts" setup>
import { computed } from 'vue'
import { getday } from '@/utils/date'
import { ElCard, ElIcon } from 'element-plus';
// Define props
const props = defineProps<{
  overviewData: {
    turnover: number
    validOrderCount: number
    orderCompletionRate: number
    unitPrice: number
    newUsers: number
  }
}>()

// Computed property for days
const days = computed(() => getday())
</script>