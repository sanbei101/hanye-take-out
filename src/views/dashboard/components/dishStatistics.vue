<template>
  <div class="container">
    <div class="homeTitle">
      <h2>
        菜品总览
      </h2>
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
      <el-card class="overviewCard" shadow="hover" style="flex-grow: 0.5;">
        <template #header>
          订单完成率
        </template>
        <div class="card-content">
          <router-link to="/dish/add" style="display: flex;align-items: center;">
            <el-icon>
              <CirclePlus />
            </el-icon>
            <span>新增菜品</span>
          </router-link>
        </div>
      </el-card>
    </div>
  </div>
</template>


<script lang="ts" setup>
import { ElCard, ElIcon } from 'element-plus';

const props = defineProps<{
  dishesData: {
    sold: number,
    discontinued: number
  }
}>()
</script>
