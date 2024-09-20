<template>
  <div class="container">
    <div class="homeTitle">
      <h2>
        套餐总览
      </h2>
    </div>
    <div class="overviewBox">
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          已启售
        </template>
        <div class="card-content">¥ {{ props.setMealData.sold }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover">
        <template #header>
          已停售
        </template>
        <div class="card-content">{{ props.setMealData.discontinued }}</div>
      </el-card>
      <el-card class="overviewCard" shadow="hover" style="flex-grow: 0.6;">
        <template #header>
          新增套餐
        </template>
        <div class="card-content">
          <router-link to="setmeal/add" style="display: flex;align-items: center;">
            <el-icon>
              <CirclePlus />
            </el-icon>
            <span>新增套餐</span>
          </router-link>
        </div>
      </el-card>
    </div>
  </div>
</template>


<script setup lang="ts">
import { ElCard, ElIcon } from 'element-plus';
interface SetMealData {
  sold: number;
  discontinued: number;
}

const props = defineProps<{
  setMealData: SetMealData;
}>()

</script>
