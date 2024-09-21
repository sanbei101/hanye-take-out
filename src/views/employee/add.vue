<script lang="ts" setup>
import { reactive, ref } from 'vue';
import { addEmployeeAPI } from '@/api/employee';
import { useRouter } from 'vue-router';
import { ElMessage, ElButton, ElCard, ElForm, ElFormItem, ElInput, ElSelect, ElOption, ElIcon, ElUpload } from 'element-plus';
import { UploadFile } from 'element-plus';

// ------ 数据 ------
const formLabelWidth = '60px';
const form = reactive({
  id: 0,
  name: '',
  account: '',
  password: '',
  phone: '',
  age: '',
  gender: '',
  pic: ''
});
const genders = [
  {
    value: 1,
    label: '男'
  },
  {
    value: 0,
    label: '女'
  }
];
const addRef = ref();

// 表单校验
const checkAge = (_rule: any, value: string, callback: (error?: Error) => void) => {
  if (value === '' || value === undefined) {
    callback(new Error('请输入年龄'));
  } else if (!/^\d+$/.test(value)) {
    callback(new Error('年龄必须为数字'));
  } else {
    const age = parseInt(value);
    if (age < 3) {
      callback(new Error('年龄不能小于3岁'));
    } else if (age > 99) {
      callback(new Error('年龄不能大于99岁'));
    } else {
      callback();
    }
  }
};
const rules = {
  name: [
    { required: true, trigger: 'blur', message: '不能为空' },
    { min: 2, message: '姓名长度不能少于2个字符', trigger: 'blur' },
    { max: 20, message: '姓名长度不能超过20个字符', trigger: 'blur' }
  ],
  account: [
    { required: true, trigger: 'blur', message: '不能为空' },
    { pattern: /^[a-zA-Z0-9]{1,10}$/, message: '用户名必须是1-10的字母数字', trigger: 'blur' }
  ],
  password: [
    { required: true, trigger: 'blur', message: '不能为空' },
    { pattern: /^\S{6,15}$/, message: '密码必须是6-15的非空字符', trigger: 'blur' }
  ],
  phone: [
    { required: true, trigger: 'blur', message: '不能为空' },
    { pattern: /^1[3-9]\d{9}$/, message: '手机号格式不正确', trigger: 'blur' }
  ],
  age: [
    { required: true, trigger: 'blur', message: '不能为空' },
    { validator: checkAge, trigger: 'blur' }
  ],
  gender: [{ required: true, trigger: 'blur', message: '不能为空' }]
};

// ------ 方法 ------

const router = useRouter();

const onFileChange = (uploadFile: UploadFile) => {
  const file = uploadFile.raw;
  if (file instanceof File) {
    const fr = new FileReader();
    fr.readAsDataURL(file);
    fr.onload = () => {
      form.pic = fr.result as string;
      console.log('头像已更新:', form.pic);
    };
  }
};

// 添加员工信息后提交（只有管理员才能对其他员工进行修改,否则普通员工只能对自己进行修改）
const submit = async () => {
  try {
    const valid = await addRef.value.validate();
    if (valid) {
      console.log('submit');
      console.log(form);
      // 在这里执行表单提交操作
      const res = await addEmployeeAPI(form);
      if (res.data.code !== 0) {
        console.log('新增员工失败！');
        return false;
      }
      // 然后进行 消息提示,页面跳转 等操作
      ElMessage({
        message: '新增员工成功',
        type: 'success'
      });
      router.push({
        path: '/employee'
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
    path: '/employee'
  });
};
</script>

<template>
  <h1>添加员工页</h1>
  <el-card>
    <el-form :model="form" :rules="rules">
      <el-form-item label="姓名" :label-width="formLabelWidth" prop="name">
        <el-input v-model="form.name" autocomplete="off" />
      </el-form-item>
      <el-form-item label="账号" :label-width="formLabelWidth" prop="account">
        <el-input v-model="form.account" autocomplete="off" />
      </el-form-item>
      <el-form-item label="密码" :label-width="formLabelWidth" prop="password">
        <el-input v-model="form.password" autocomplete="off" />
      </el-form-item>
      <el-form-item label="电话" :label-width="formLabelWidth" prop="phone">
        <el-input v-model="form.phone" autocomplete="off" />
      </el-form-item>
      <el-form-item label="年龄" :label-width="formLabelWidth" prop="age">
        <el-input v-model="form.age" autocomplete="off" />
      </el-form-item>
      <el-form-item label="性别" :label-width="formLabelWidth" prop="gender">
        <el-select clearable v-model="form.gender" placeholder="选择分类类型">
          <el-option v-for="item in genders" :key="item.value" :label="item.label" :value="item.value" />
        </el-select>
      </el-form-item>
      <el-form-item label="头像" :label-width="formLabelWidth" prop="pic">
        <el-upload :show-file-list="false" :auto-upload="false" accept="image/*" :on-change="onFileChange">
          <el-button type="primary">
            <el-icon style="font-size: 15px; margin-right: 10px">
              <Plus />
            </el-icon>
            选择图片
          </el-button>
        </el-upload>
      </el-form-item>
    </el-form>
    <el-form-item>
      <el-button class="submit_btn" type="success" @click="submit">添加</el-button>
      <el-button class="cancel_btn" type="info" plain @click="cancel">取消</el-button>
    </el-form-item>
  </el-card>
</template>
