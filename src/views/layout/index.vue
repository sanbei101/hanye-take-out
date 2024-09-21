<script setup lang="ts" name="layout">
import { RouterView, useRouter } from 'vue-router';
import { ElMessageBox, ElMessage, ElDialog, ElRadioGroup, ElRadio, ElButton, ElForm, ElFormItem, ElInput, ElTag } from 'element-plus';
import {
  ElContainer,
  ElHeader,
  ElIcon,
  ElDropdown,
  ElDropdownItem,
  ElDropdownMenu,
  ElMenuItem,
  ElMenu,
  ElMain,
  ElSpace
} from 'element-plus';
import { ElNotification } from 'element-plus';
import { useUserInfoStore } from '@/store';
import { ref, reactive, onMounted, onBeforeUnmount } from 'vue';
import { fixPwdAPI } from '@/api/employee';
import { getStatusAPI, fixStatusAPI } from '@/api/shop';

// ------ data ------
const dialogFormVisible = ref(false);
const dialogStatusVisible = ref(false);
const formLabelWidth = '80px';
const isCollapse = ref(false);

const menuList = [
  {
    title: '控制台',
    path: '/dashboard',
    icon: 'pieChart'
  },
  {
    title: '数据统计',
    path: '/statistics',
    icon: 'memo'
  },
  {
    title: '订单管理',
    path: '/order',
    icon: 'collection'
  },
  {
    title: '分类管理',
    path: '/category',
    icon: 'postcard'
  },
  {
    title: '套餐管理',
    path: '/setmeal',
    icon: 'user'
  },
  {
    title: '菜品管理',
    path: '/dish',
    icon: 'dish'
  },
  {
    title: '员工管理',
    path: '/employee',
    icon: 'setting'
  }
];

const form = reactive({
  oldPwd: '',
  newPwd: '',
  rePwd: ''
});
const status = ref(1);
const status_active = ref(1); // 单选框绑定的动态值

// ------ method ------
const router = useRouter();
const userInfoStore = useUserInfoStore();

// 初始化时获取营业状态
const init = async () => {
  const { data: res } = await getStatusAPI();
  console.log('初始化后的status status_active', res.data);
  status.value = res.data;
  status_active.value = res.data;
};
init();

// 关闭修改店铺状态对话框
const cancelStatus = () => {
  ElMessage({
    type: 'info',
    message: '已取消修改'
  });
  dialogStatusVisible.value = false;
};
// 关闭修改密码对话框
const cancelForm = () => {
  ElMessage({
    type: 'info',
    message: '已取消修改'
  });
  dialogFormVisible.value = false;
};
// 修改店铺状态
const fixStatus = async () => {
  console.log('修改后的店铺状态为');
  console.log(status_active.value);
  const { data: res } = await fixStatusAPI(status_active.value);
  if (res.code != 0) return; // 修改失败信息会在相应拦截器中捕获并提示
  // 修改成功才改变status的值
  status.value = status_active.value;
  ElMessage({
    type: 'success',
    message: '修改成功'
  });
  dialogStatusVisible.value = false;
};
// 修改密码
const changePassword = async () => {
  const submitForm = {
    oldPwd: form.oldPwd,
    newPwd: form.newPwd
  };
  console.log('要提交的表单信息');
  console.log(submitForm);
  const { data: res } = await fixPwdAPI(submitForm);
  if (res.code != 0) return; // 密码错误信息会在相应拦截器中捕获并提示
  ElMessage({
    type: 'success',
    message: '修改成功'
  });
  dialogFormVisible.value = false;
};

const logoutUser = () => {
  // 为了让用户体验更好，来个确认提示框
  ElMessageBox.confirm('走了，爱是会消失的吗?', '退出登录', {
    confirmButtonText: 'OK',
    cancelButtonText: 'Cancel',
    type: 'warning'
  })
    .then(() => {
      ElMessage({
        type: 'success',
        message: '退出成功'
      });
      // 清除用户信息，包括token
      userInfoStore.userInfo = null;
      console.log(userInfoStore);
      router.push('/login');
    })
    .catch(() => {
      ElMessage({
        type: 'info',
        message: '已取消退出'
      });
    });
};

// refs
const websocket = ref<WebSocket | null>(null);
const shopShow = ref(false);

const audio1 = ref<HTMLAudioElement | null>(null);
const audio2 = ref<HTMLAudioElement | null>(null);

const webSocket = () => {
  const clientId = Math.random().toString(36).slice(2);
  const socketUrl = 'ws://localhost:8081/ws/' + clientId;
  console.log('socketUrl', socketUrl);

  if (typeof WebSocket == 'undefined') {
    console.log('当前浏览器无法接收实时报警信息，请使用谷歌浏览器！');
    ElNotification({
      title: '提示',
      message: '当前浏览器无法接收实时报警信息，请使用谷歌浏览器！',
      type: 'warning',
      duration: 0
    });
  } else {
    websocket.value = new WebSocket(socketUrl);
    websocket.value.onopen = () => {
      console.log('浏览器WebSocket已打开');
    };
    websocket.value.onmessage = (msg) => {
      console.log('接收到的消息', msg);
      audio1.value && audio1.value.click();
      // 重置音频，从头开始播放
      audio1.value!.currentTime = 0;
      audio2.value!.currentTime = 0;
      // 解析服务器通过WebSocket发送的消息
      const jsonMsg = JSON.parse(msg.data);
      if (jsonMsg.type === 1) {
        audio1.value!.play();
      } else if (jsonMsg.type === 2) {
        audio2.value!.play();
      }
      // 右上角弹窗提示
      ElNotification({
        title: jsonMsg.type === 1 ? '待接单' : '催单',
        message:
          jsonMsg.type === 1
            ? `<span>您有1个<span style="color:#419EFF">订单待处理</span>,${jsonMsg.content},请及时接单</span>`
            : `${jsonMsg.content}<span style='color:#419EFF;cursor: pointer'>去处理</span>`,
        duration: 0,
        dangerouslyUseHTMLString: true,
        onClick: () => {
          router.push(`/order?orderId=${jsonMsg.orderId}`).catch((err) => {
            console.log(err);
          });
          setTimeout(() => {
            location.reload();
          }, 100);
        }
      });
    };
    websocket.value.onerror = () => {
      ElNotification({
        title: '错误',
        message: '服务器错误，无法接收实时报警信息',
        type: 'error',
        duration: 0
      });
    };
    websocket.value.onclose = () => {
      console.log('WebSocket已关闭');
    };
  }
};

const handleClose = () => {
  shopShow.value = false;
};

// lifecycle hooks
onMounted(() => {
  document.addEventListener('click', handleClose);
  // getStatus()
  webSocket();
});

onBeforeUnmount(() => {
  if (websocket.value) {
    websocket.value.close();
  }
});
</script>

<template>
  <div class="common-layout">
    <el-dialog v-model="dialogStatusVisible" title="店铺状态设置" width="700">
      <el-radio-group v-model="status_active">
        <el-radio :value="1" size="large"
          >营业中
          <span>当前餐厅处于营业状态，自动接收任何订单，可点击打烊进入店铺打烊状态。</span>
        </el-radio>
        <el-radio :value="0" size="large"
          >打烊中
          <span>当前餐厅处于打烊状态，仅接受营业时间内的预定订单，可点击营业中手动恢复营业状态。</span>
        </el-radio>
      </el-radio-group>
      <template #footer>
        <el-button @click="cancelStatus">取消</el-button>
        <el-button type="primary" @click="fixStatus">确定</el-button>
      </template>
    </el-dialog>
    <el-dialog v-model="dialogFormVisible" title="修改密码" width="500">
      <el-form :model="form">
        <el-form-item prop="oldPwd" label="原密码" :label-width="formLabelWidth">
          <el-input v-model="form.oldPwd" autocomplete="off" />
        </el-form-item>
        <el-form-item prop="newPwd" label="新密码" :label-width="formLabelWidth">
          <el-input v-model="form.newPwd" autocomplete="off" />
        </el-form-item>
        <el-form-item prop="rePwd" label="确认密码" :label-width="formLabelWidth">
          <el-input v-model="form.rePwd" autocomplete="off" />
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="cancelForm">取消</el-button>
        <el-button type="primary" @click="changePassword">确定</el-button>
      </template>
    </el-dialog>

    <el-container>
      <el-header style="display: flex; justify-content: space-between; align-items: center">
        <el-space>
          <el-icon v-if="isCollapse" style="cursor: pointer">
            <Expand @click.stop="isCollapse = !isCollapse" />
          </el-icon>
          <el-icon v-else style="cursor: pointer">
            <Fold @click.stop="isCollapse = !isCollapse" />
          </el-icon>
          <el-tag type="primary">{{ status == 1 ? '营业中' : '打烊中' }}</el-tag>
        </el-space>
        <el-space>
          <el-dropdown>
            <el-button type="primary">
              {{ userInfoStore.userInfo ? userInfoStore.userInfo.account : '未登录' }}
              <el-icon class="arrow-down-icon"><arrow-down /></el-icon>
            </el-button>
            <template #dropdown>
              <el-dropdown-menu>
                <el-dropdown-item @click="dialogFormVisible = true">修改密码</el-dropdown-item>
                <el-dropdown-item @click="logoutUser">退出登录</el-dropdown-item>
              </el-dropdown-menu>
            </template>
          </el-dropdown>
          <el-button @click="dialogStatusVisible = true">店铺状态设置</el-button>
        </el-space>
      </el-header>

      <el-container>
        <el-menu :width="isCollapse ? '640px' : '200px'" :collapse="isCollapse" unique-opened router>
          <template v-for="item in menuList" :key="item.path">
            <el-menu-item :index="item.path">
              <el-icon>
                <component :is="item.icon" />
              </el-icon>
              <span>{{ item.title }}</span>
            </el-menu-item>
          </template>
        </el-menu>

        <el-container>
          <el-main>
            <router-view></router-view>
          </el-main>
        </el-container>
      </el-container>
    </el-container>
  </div>
</template>
