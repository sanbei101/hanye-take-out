<script lang="ts" setup>
import { reactive, ref, watch } from 'vue';
import { addDishAPI, getDishByIdAPI, updateDishAPI } from '@/api/dish';
import { getCategoryPageListAPI } from '@/api/category';
import { useRouter, useRoute } from 'vue-router';
import { ElMessage, ElCard, ElForm, ElFormItem, ElInput, ElButton, ElIcon, UploadFile } from 'element-plus';

interface Category {
  id: number;
  name: string;
}

// 菜品id对应的分类列表,即categoryId字段不能只展示id值,应该根据id查询到对应的分类名进行回显
const categoryList = ref<Category[]>([]);
const formLabelWidth = '70px';

const form = reactive({
  id: 0,
  name: '',
  pic: '',
  detail: '',
  price: '',
  status: '',
  categoryId: ''
});
const count = ref(0);
// 图片下的隐藏input框
const addRef = ref();

// 表单校验
const rules = {
  name: [{ required: true, trigger: 'blur', message: '不能为空' }],
  detail: [{ required: true, trigger: 'blur', message: '不能为空' }],
  price: [{ required: true, trigger: 'blur', message: '不能为空' }],
  categoryId: [{ required: true, trigger: 'blur', message: '不能为空' }]
};

// ------ 方法 ------

const router = useRouter();
const route = useRoute();

// 在文件管理器中选择图片后触发的改变事件:预览
const onFileChange = (uploadFile: UploadFile) => {
  const file = uploadFile.raw;
  if (file instanceof File) {
    const fr = new FileReader();
    fr.readAsDataURL(file);
    fr.onload = () => {
      form.pic = fr.result as string;
      console.log('菜品图片已更新:', form.pic);
    };
  }
};

const change = () => {
  console.log('watch 函数监听 count 变化,执行相应change回调 - getLeftDishFlavor');
};
watch(count, change);

// 添加菜品信息后提交
const submit = async (keep: any) => {
  const valid = await addRef.value.validate();
  if (valid) {
    let params: any = { ...form };
    console.log('看看有没有拷贝成功？', params);
    // 需要先对口味数组进行json.stringfy序列化

    delete params.dishFlavors;
    console.log('看看有没有serialize成功？', params);
    // --- 处理完毕,开始提交 ---
    // 情况1:无路径参数,form.id保持默认值0,新增菜品
    if (form.id === 0) {
      console.log('新增菜品');
      const res = await addDishAPI(params);
      if (res.data.code !== 0) {
        console.log('新增菜品失败！');
        return false;
      }
      ElMessage({
        message: '新增菜品成功',
        type: 'success'
      });
      // 根据keep决定是否继续添加
      if (keep) {
        form.id = 0;
        form.name = '';
        form.pic = '';
        form.detail = '';
        form.price = '';
        form.status = '';
        form.categoryId = '';
      } else {
        router.push({
          path: '/dish'
        });
      }
    }
    // 情况2:有路径参数,修改菜品
    else {
      console.log('修改菜品');
      const res = await updateDishAPI(params);
      if (res.data.code !== 0) {
        console.log('修改菜品失败！');
        return false;
      }
      ElMessage({
        message: '修改菜品成功',
        type: 'success'
      });
      router.push({
        path: '/dish'
      });
    }
  } else {
    console.log('form not valid!');
    return false;
  }
};
// 取消修改
const cancel = () => {
  router.push({
    path: '/dish'
  });
};

const init = async () => {
  // 1. 获取菜品分类,等下菜品选择分类时有个下拉框,要展示所有菜品的分类
  // 由于复用分页查询的API,这里不需要分页且数据量较少,所以pageSize设置为100
  const { data: res } = await getCategoryPageListAPI({ page: 1, pageSize: 100, type: 1 });
  console.log('分类列表');
  console.log(res.data);
  categoryList.value = res.data.records;
  console.log('categoryList: ', categoryList.value);
  // 2. 由于当前页面可能是add也可能是update,所以要根据路由参数来判断是否需要getDishById
  if (route.query.id !== undefined) {
    console.log('修改菜品页');
    form.id = route.query.id ? parseInt(route.query.id as string) : 0;
    let dish = await getDishByIdAPI(form.id);
    console.log(dish);
    Object.assign(form, dish.data.data);
    console.log(form);
  } else {
    console.log('新增菜品页', form.id);
  }
};

init();
</script>

<template>
  <h1>添加菜品页</h1>
  <el-card>
    <el-form :model="form" :rules="rules" ref="addRef">
      <el-form-item label="名称" :label-width="formLabelWidth" prop="name">
        <el-input v-model="form.name" autocomplete="off" />
      </el-form-item>
      <el-form-item label="图片" :label-width="formLabelWidth" prop="pic">
        <el-upload :show-file-list="false" :auto-upload="false" accept="image/*" :on-change="onFileChange">
          <el-button type="primary">
            <el-icon style="font-size: 15px; margin-right: 10px">
              <Plus />
            </el-icon>
            选择图片
          </el-button>
        </el-upload>
      </el-form-item>

      <el-form-item label="详情" :label-width="formLabelWidth" prop="detail">
        <el-input v-model="form.detail" autocomplete="off" type="textarea" />
      </el-form-item>
      <el-form-item label="价格" :label-width="formLabelWidth" prop="price">
        <el-input v-model="form.price" autocomplete="off" />
      </el-form-item>
      <el-form-item label="分类" :label-width="formLabelWidth" prop="categoryId">
        <el-select clearable v-model="form.categoryId" placeholder="选择分类类型">
          <el-option v-for="item in categoryList" :key="item.id" :label="item.name" :value="item.id" />
        </el-select>
      </el-form-item>
    </el-form>
    <el-form-item>
      <el-button class="submit_btn" type="success" @click="submit(0)">保存并退出</el-button>
      <el-button v-if="form.id == 0" class="continue_btn" type="success" plain @click="submit(1)">保存并继续添加</el-button>
      <el-button class="cancel_btn" type="info" plain @click="cancel">取消</el-button>
    </el-form-item>
  </el-card>
</template>
