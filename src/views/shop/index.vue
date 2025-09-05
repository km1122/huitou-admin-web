<template>
  <div class="table-box">
    <ProTable ref="proTable" title="菜单列表" row-key="path" :indent="20" :columns="columns" :data="shopData">
      <!-- 表格 header 按钮 -->
      <template #tableHeader>
        <el-button type="primary" :icon="CirclePlus" @click="generateQR">新增店铺授权</el-button>
        <el-dialog v-model="showQRDialog" title="请扫描二维码" width="400px" center>
          <div class="qr-content">
            <transition name="el-fade-in">
              <img v-show="qrImage" :src="qrImage" class="qr-image" />
            </transition>
            <div v-if="!qrImage" class="loading-wrapper">
              <el-icon :size="40" class="is-loading">
                <Loading />
              </el-icon>
            </div>
          </div>

          <template #footer>
            <div class="dialog-footer">
              <el-button type="primary" @click="confirmScan" :loading="confirming">我已经登录</el-button>
              <el-button @click="showQRDialog = false">关闭</el-button>
            </div>
          </template>
        </el-dialog>
      </template>
      <!-- 菜单图标 -->
      <template #icon="scope">
        <el-icon :size="18">
          <component :is="scope.row.meta.icon"></component>
        </el-icon>
      </template>
      <!-- 菜单操作 -->
      <template #operation>
        <el-button type="primary" link :icon="Refresh"> 重新授权 </el-button>
      </template>
    </ProTable>
  </div>
</template>

<script setup lang="ts" name="shopMange">
import { ref } from "vue";
import { ColumnProps } from "@/components/ProTable/interface";
import { CirclePlus, Refresh } from "@element-plus/icons-vue";
import authShopList from "@/assets/json/authShopList.json";
import ProTable from "@/components/ProTable/index.vue";
import { ElMessage } from "element-plus";
import loginQrCode from "@/assets/json/getLoginQrCode.json";
import checkLoginQrCode from "@/assets/json/checkLoginQrCode.json";

const proTable = ref();

const shopData = ref(authShopList.data);

// 表格配置项
const columns: ColumnProps[] = [
  { prop: "name", label: "店铺名称", align: "left", search: { el: "input" } },
  { prop: "loginStatus", label: "登陆状态" },
  { prop: "loginTime", label: "登陆时间" },
  { prop: "loginUser", label: "操作人", width: 300 },
  { prop: "operation", label: "操作", width: 250, fixed: "right" }
];

// 响应式数据
const showQRDialog = ref(false);
const qrImage = ref("");
const qrTicket = ref("");
const generating = ref(false);
const confirming = ref(false);

const generateQR = async () => {
  try {
    generating.value = true;
    //const { data } = await axios.get("/api/qrcode");
    // 调用后端接口
    /*let response = await axios.post("/api/qrcode", {
      scene: "login",
      expire: 300 // 5分钟过期
    });*/
    let response = loginQrCode;
    if (response.code != 200) {
      ElMessage.success("二维码生成失败");
      generating.value = false;
      return;
    }
    qrImage.value = `data:image/png;base64,${response.data.base64Image}`;
    showQRDialog.value = true;
    qrTicket.value = response.data.ticket;
  } catch (error) {
    ElMessage.error("二维码生成失败");
  } finally {
    generating.value = false;
  }
};

const confirmScan = async () => {
  confirming.value = true;
  try {
    //await axios.post("/api/confirm-scan");
    /*let response = await axios.get("/api/confirm-scan", {
      params: { token: qrTicket.value }
    });*/
    let response = checkLoginQrCode;
    if (response.code == 0) {
      ElMessage.success("登陆成功");
      showQRDialog.value = false;
    } else {
      ElMessage.success("还没登陆");
      showQRDialog.value = true;
    }
  } catch (error) {
    ElMessage.error("登陆失败");
  } finally {
    confirming.value = false;
  }
};
</script>

<style scoped>
.qr-generator {
  padding: 20px;
}

.qr-content {
  min-height: 300px;
  display: flex;
  justify-content: center;
  align-items: center;
}

.qr-image {
  width: 100%;
  max-width: 300px;
  border: 1px solid #eee;
  border-radius: 8px;
}

.loading-wrapper {
  padding: 50px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 12px;
}
</style>
