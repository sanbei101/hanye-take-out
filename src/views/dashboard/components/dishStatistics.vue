<template>
  <div class="container">
    <div class="homeTitle">
      <h2>
        菜品总览
      </h2>
      <div class="more">
        <router-link to="statistics">菜品管理</router-link>
        <el-icon>
          <ArrowRight />
        </el-icon>
      </div>
    </div>
    <div class="overviewBox">
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          已启售
        </template>
        <div class="card-content">¥ {{ props.dishesData.sold }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          已停售
        </template>
        <div class="card-content">{{ props.dishesData.discontinued }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          订单完成率
        </template>
        <div class="card-content">
          <router-link to="/dish/add">
            <el-icon>
              <CirclePlus />
            </el-icon>
            <p>新增菜品</p>
          </router-link>
        </div>
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

// Define propsimport { computed } from 'vue'
import { ElCard, ElIcon } from 'element-plus';

const props = defineProps<{
  dishesData: {
    sold: number,
    discontinued: number
  }
}>()
</script>
