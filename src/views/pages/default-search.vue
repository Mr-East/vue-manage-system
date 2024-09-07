<template>
  <div class="search-default-applications">
    <el-card>
      <h2>查询违约记录信息</h2>

      <!-- 查询输入框 -->
      <div class="search-section">
        <el-input
          v-model="searchTerm"
          placeholder="输入用户名称进行搜索"
          style="width: 300px"
        />
        <el-button type="primary" @click="searchApplications" style="margin-left: 10px">
          搜索
        </el-button>
      </div>

      <!-- 违约记录表格 -->
      <el-table
        v-if="applications.length > 0"
        :data="applications"
        style="width: 100%"
        class="mt-4"
      >
        <el-table-column prop="id" label="违约ID" width="80" />
        <el-table-column prop="customer_name" label="客户名称" />
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="severity" label="严重性" />
        <el-table-column prop="remarks" label="备注" />
        <el-table-column prop="application_time" label="申请时间" />
        <el-table-column prop="audit_status" label="审核状态">
          <template #default="scope">
            <span v-if="scope.row.audit_status === 0">未审核</span>
            <span v-else-if="scope.row.audit_status === 1">已通过</span>
            <span v-else-if="scope.row.audit_status === 2">未通过</span>
          </template>
        </el-table-column>
      </el-table>

      <!-- 无记录显示 -->
      <el-empty
        v-if="applications.length === 0 && !loading"
        description="无违约记录"
      ></el-empty>

      <!-- 加载中提示 -->
      <div v-if="loading" class="loading">加载中...</div>
    </el-card>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import { ElMessage } from "element-plus";
import axios from "axios";

// 搜索框输入内容
const searchTerm = ref("");

// 存储查询结果
const applications = ref([]);

// 加载状态
const loading = ref(false);

// 查询违约记录信息
const searchApplications = async () => {
  loading.value = true;
  try {
    const response = await axios.get(
      "http://127.0.0.1:5000/default_applications/search",
      {
        params: {
          customer_name: searchTerm.value.trim(), // 去除前后空白
        },
      }
    );

    // 如果返回消息
    if (response.data.message) {
      ElMessage.info(response.data.message);
      applications.value = [];
    } else {
      applications.value = response.data;
    }
  } catch (error) {
    ElMessage.error("查询失败，请稍后再试");
    console.error(error);
  } finally {
    loading.value = false;
  }
};

// 页面加载时获取所有记录
onMounted(() => {
  searchApplications(); // 初次加载时显示所有记录或根据输入的名称过滤
});
</script>

<style scoped>
.search-default-applications {
  padding: 20px;
}
.search-section {
  display: flex;
  margin-bottom: 20px;
}
.loading {
  text-align: center;
  font-size: 16px;
  color: #666;
}
.mt-4 {
  margin-top: 20px;
}
</style>
