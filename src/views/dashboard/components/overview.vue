<template>
  <div class="container">
    <div class="homeTitle">
      <h2>
        今日数据
      </h2>
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



<script lang="ts" setup>
import { ElCard } from 'element-plus';
const props = defineProps<{
  overviewData: {
    turnover: number
    validOrderCount: number
    orderCompletionRate: number
    unitPrice: number
    newUsers: number
  }
}>()

</script>