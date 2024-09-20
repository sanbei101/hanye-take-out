<template>
  <div class="container">
    <div class="homeTitle">
      <h2>
        订单管理
      </h2>
    </div>
    <div class="overviewBox">
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          待接单
        </template>
        <div class="card-content">¥ {{ props.orderviewData.waitingOrders }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          待派送
        </template>
        <div class="card-content">{{ props.orderviewData.deliveredOrders }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          已完成
        </template>
        <div class="card-content">
          <router-link to="/order?status=5">
            {{ props.orderviewData.completedOrders }}
          </router-link>
        </div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          已取消
        </template>
        <div class="card-content">¥ {{ props.orderviewData.cancelledOrders }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          全部订单
        </template>
        <div class="card-content">{{ props.orderviewData.allOrders }}</div>
      </el-card>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ElCard } from 'element-plus';

// Define props
const props = defineProps<{
  orderviewData: {
    waitingOrders: number,
    deliveredOrders: number,
    completedOrders: number,
    cancelledOrders: number,
    allOrders: number
  }
}>()
</script>
