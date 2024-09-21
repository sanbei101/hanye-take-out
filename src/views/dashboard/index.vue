<template>
  <div class="dashboard-container home">
    <!-- 营业数据 -->
    <Overview :overviewData="overviewData" />
    <!-- end -->
    <!-- 订单管理 -->
    <Orderview :orderviewData="orderviewData" />
    <!-- end -->
    <div class="homeMain">
      <!-- 菜品总览 -->
      <CuisineStatistics :dishesData="dishesData" />
      <!-- end -->
      <!-- 套餐总览 -->
      <SetMealStatistics :setMealData="setMealData" />
      <!-- end -->
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted } from 'vue';
import { getBusinessDataAPI, getOrderDataAPI, getOverviewDishesAPI, getSetMealStatisticsAPI } from '@/api/dashboard';
import { getOrderListByAPI } from '@/api/order';
import Overview from './components/overview.vue';
import Orderview from './components/orderview.vue';
import CuisineStatistics from './components/dishStatistics.vue';
import SetMealStatistics from './components/setmealStatistics.vue';

// 状态定义
const overviewData = ref<any>({});
const orderviewData = ref<any>({});
const dishesData = ref<any>({});
const setMealData = ref<any>({});
const orderStatics = ref<any>({});

const init = async () => {
  try {
    const businessData = await getBusinessDataAPI();
    overviewData.value = businessData.data.data;

    const orderData = await getOrderDataAPI();
    orderviewData.value = orderData.data.data;

    const overviewDishes = await getOverviewDishesAPI();
    dishesData.value = overviewDishes.data.data;

    const setMealStatistics = await getSetMealStatisticsAPI();
    setMealData.value = setMealStatistics.data.data;

    await getOrderListBy3Status();
  } catch (error) {
    console.error('初始化数据时出错: ', error);
  }
};

// 获取待处理,待派送,派送中数量
const getOrderListBy3Status = async () => {
  try {
    const res = await getOrderListByAPI();
    if (res.data.code === 0) {
      orderStatics.value = res.data.data;
    } else {
      console.error(res.data.msg);
    }
  } catch (err) {
    console.error('请求出错了: ', err);
  }
};

onMounted(() => {
  init();
});
</script>

<style>
.homeTitle {
  display: flex;
  align-items: center;
  justify-content: center;
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
  display: flex;
  justify-content: center;
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
