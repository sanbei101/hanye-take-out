<script lang="ts" setup>
import { reactive, ref } from 'vue';
import { addCategoryAPI } from '@/api/category';
import { useRouter } from 'vue-router';
import { ElMessage, ElCard, ElForm, ElFormItem, ElInput, ElSelect, ElButton } from 'element-plus';

// ------ 数据 ------
const formLabelWidth = '60px';
const form = reactive({
  id: 0,
  name: '',
  type: '',
  sort: ''
});
const options = [
  {
    value: '1',
    label: '菜品分类'
  },
  {
    value: '2',
    label: '套餐分类'
  }
];

const addRef = ref();

// 表单校验
const rules = {
  name: [{ required: true, trigger: 'blur', message: '不能为空' }],
  type: [{ required: true, trigger: 'blur', message: '不能为空' }],
  sort: [{ required: true, trigger: 'blur', message: '不能为空' }]
};

// ------ 方法 ------

const router = useRouter();

// 添加分类信息后提交（只有管理员才能对其他分类进行修改,否则普通分类只能对自己进行修改）
const submit = async () => {
  try {
    const valid = await addRef.value.validate();
    if (valid) {
      console.log('submit');
      console.log(form);
      // 在这里执行表单提交操作
      const res = await addCategoryAPI(form);
      if (res.data.code !== 0) {
        console.log('新增分类失败！');
        return false;
      }
      // 然后进行 消息提示,页面跳转 等操作
      ElMessage({
        message: '新增分类成功',
        type: 'success'
      });
      router.push({
        path: '/category'
      });
    } else {
      console.log('form not valid!');
      return false;
    }
  } catch (error) {
    console.error('执行过程中失败:', error);
  }
};
// 取消修改
const cancel = () => {
  router.push({
    path: '/category'
  });
};
</script>

<template>
  <h1>添加分类页</h1>
  <el-card>
    <el-form :model="form" :rules="rules" ref="addRef">
      <el-form-item label="名称" :label-width="formLabelWidth" prop="name">
        <el-input v-model="form.name" autocomplete="off" />
      </el-form-item>
      <el-form-item label="类型" :label-width="formLabelWidth" prop="type">
        <el-select clearable v-model="form.type" placeholder="选择分类类型">
          <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value" />
        </el-select>
      </el-form-item>
      <el-form-item label="排序" :label-width="formLabelWidth" prop="sort">
        <el-input v-model="form.sort" autocomplete="off" />
      </el-form-item>
    </el-form>
    <el-form-item>
      <el-button class="submit_btn" type="success" @click="submit">添加</el-button>
      <el-button class="cancel_btn" type="info" plain @click="cancel">取消</el-button>
    </el-form-item>
  </el-card>
</template>
