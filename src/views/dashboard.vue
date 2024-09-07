<template>
  <div>
    <template v-if="username === 'admin'">
      <el-row :gutter="20" class="mgb20">
        <el-col :span="12">
          <el-card shadow="hover">
            <div class="card-header">
              <p class="card-header-title">用户地区分布</p>
              <p class="card-header-desc">展示不同地区的用户数量</p>
            </div>
            <v-chart class="chart" :option="regionChartOption" />
          </el-card>
        </el-col>
        <el-col :span="12">
          <el-card shadow="hover">
            <div class="card-header">
              <p class="card-header-title">行业统计</p>
              <p class="card-header-desc">展示不同行业的用户数量</p>
            </div>
            <v-chart class="chart" :option="industryChartOption" />
          </el-card>
        </el-col>
      </el-row>
      <el-row :gutter="20" class="mgb20">
        <el-col :span="24">
          <el-card shadow="hover">
            <div class="card-header">
              <p class="card-header-title">查询用户信息</p>
              <p class="card-header-desc">通过用户名称查询用户信息</p>
            </div>
            <div class="search-section">
              <el-input
                v-model="searchTerm"
                placeholder="输入用户名称进行搜索"
              ></el-input>
              <el-button type="primary" @click="searchCustomers">搜索</el-button>
            </div>
            <div v-if="loading">加载中...</div>
            <div v-if="search_error" class="error">{{ Search_error }}</div>
            <el-table v-if="customers.length > 0" :data="customers" style="width: 100%">
              <el-table-column prop="customer_id" label="用户ID"></el-table-column>
              <el-table-column prop="customer_name" label="用户名称"></el-table-column>
              <el-table-column prop="username" label="用户名"></el-table-column>
              <el-table-column
                prop="industry_classification"
                label="行业分类"
              ></el-table-column>
              <el-table-column
                prop="region_classification"
                label="地区分类"
              ></el-table-column>
              <el-table-column prop="credit_rating" label="信用评级"></el-table-column>
              <el-table-column prop="group" label="用户组"></el-table-column>
              <el-table-column prop="external_rating" label="外部评级"></el-table-column>
            </el-table>
          </el-card>
        </el-col>
      </el-row>
      <el-row :gutter="20" class="mgb20">
        <el-col :span="24">
          <el-card shadow="hover">
            <div class="card-header">
              <p class="card-header-title">删除违约记录</p>
              <p class="card-header-desc">通过违约申请记录ID删除记录</p>
            </div>
            <div class="delete-section">
              <el-input v-model="deleteId" placeholder="输入违约申请记录ID"></el-input>
              <el-button type="danger" @click="deleteDefaultApplication"
                >删除记录</el-button
              >
            </div>
            <div v-if="deleteLoading">删除中...</div>
            <div v-if="deleteError" class="error">{{ deleteError }}</div>
          </el-card>
        </el-col>
      </el-row>
      <el-row :gutter="20" class="mgb20">
        <el-col :span="24">
          <el-card shadow="hover">
            <div class="card-header">
              <p class="card-header-title">审核违约重生申请</p>
            </div>
            <div class="review-section">
              <el-button @click="fetchPendingRebirths">加载待审核申请</el-button>
              <div v-if="loading">加载中...</div>
              <div v-if="error" class="error">{{ error }}</div>
              <el-table
                v-if="pendingRebirths.length > 0"
                :data="pendingRebirths"
                style="width: 100%"
              >
                <el-table-column prop="id" label="申请ID"></el-table-column>
                <el-table-column prop="customer_id" label="客户ID"></el-table-column>
                <el-table-column prop="default_id" label="违约ID"></el-table-column>
                <el-table-column prop="audit_status" label="审核状态"></el-table-column>
                <el-table-column label="操作">
                  <template #default="{ row }">
                    <el-button type="success" @click="submitReview(row.id, 1)"
                      >通过</el-button
                    >
                    <el-button type="danger" @click="submitReview(row.id, 2)"
                      >驳回</el-button
                    >
                  </template>
                </el-table-column>
              </el-table>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </template>
    <template v-else>

    <el-card shadow="always" :body-style="{ padding: '20px' }">
    <template #header>
    <div>
    <span>{{wenhou +',' +  username}}</span>
    </div>
    </template>
    欢迎来到违约客户管理系统
    </el-card>
    
        
    </template>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import countup from "@/components/countup.vue";
import axios from "axios";
import { use, registerMap } from "echarts/core";
import { PieChart } from "echarts/charts";
import {
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent,
} from "echarts/components";
import { CanvasRenderer } from "echarts/renderers";
import VChart from "vue-echarts";
import { dashOpt1, dashOpt2, mapOptions } from "./chart/options";
import chinaMap from "@/utils/china";
import { useUserStore } from "@/store/user";
const userStore = useUserStore();
import {time} from "@/utils/time";
const username: string | null = localStorage.getItem('vuems_name');
let wenhou = time();
use([
  CanvasRenderer,
  PieChart,
  GridComponent,
  TooltipComponent,
  LegendComponent,
  TitleComponent,
]);

// 注册中国地图
registerMap("china", chinaMap);

// 定义饼状图的响应式数据
const regionChartOption = ref({});
const industryChartOption = ref({});
const customers = ref([]);
const searchTerm = ref("");
const pendingRebirths = ref([]);
const loading = ref(false);
const search_error = ref(null);
// Define reactive data for delete operation
const deleteId = ref("");
const deleteLoading = ref(false);
const deleteError = ref(null);
const error = ref(null);
const Search_error = ref(null);
// Fetch data for region statistics

const fetchRegionData = async () => {
  try {
    const response = await axios.get("http://127.0.0.1:5000/statistics/region");
    const data = response.data.map((item) => ({
      value: item.default_count,
      name: item.region,
    }));
    regionChartOption.value = {
      title: {
        text: "用户地区分布",
        subtext: "不同地区的用户数量",
        left: "center",
        textStyle: {
          color: "#fff",
        },
      },
      tooltip: {
        trigger: "item",
        formatter: "{a} <br/>{b} : {c} ({d}%)",
      },
      series: [
        {
          name: "地区",
          type: "pie",
          radius: "50%",
          center: ["50%", "60%"],
          data: data,
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: "rgba(0, 0, 0, 0.5)",
            },
          },
        },
      ],
    };
  } catch (error) {
    console.error("Error fetching region data:", error);
  }
};

// Fetch data for industry statistics
const fetchIndustryData = async () => {
  try {
    const response = await axios.get("http://127.0.0.1:5000/statistics/industry");
    const data = response.data.map((item) => ({
      value: item.default_count,
      name: item.industry,
    }));
    industryChartOption.value = {
      title: {
        text: "行业统计",
        subtext: "不同行业的用户数量",
        left: "center",
        textStyle: {
          color: "#fff",
        },
      },
      tooltip: {
        trigger: "item",
        formatter: "{a} <br/>{b} : {c} ({d}%)",
      },
      series: [
        {
          name: "行业",
          type: "pie",
          radius: "50%",
          center: ["50%", "60%"],
          data: data,
          emphasis: {
            itemStyle: {
              shadowBlur: 10,
              shadowOffsetX: 0,
              shadowColor: "rgba(0, 0, 0, 0.5)",
            },
          },
        },
      ],
    };
  } catch (error) {
    console.error("Error fetching industry data:", error);
  }
};
const searchCustomers = async () => {
  loading.value = true;
  customers.value = [];
  search_error.value = null;
  try {
    // 依赖 axios 自动处理 params 选项
    const response = await axios.get("http://127.0.0.1:5000/api/search_customers", {
      params: { customer_name: searchTerm.value },
    });
    customers.value = response.data.customers;
  } catch (err) {
    console.error(err);
    if (err.response && err.response.status === 404) {
      Search_error.value = "未找到用户";
    } else {
      Search_error.value = "搜索出错";
    }
  } finally {
    loading.value = false;
  }
};

// Delete default application by ID
const deleteDefaultApplication = async () => {
  deleteLoading.value = true;
  deleteError.value = null;

  try {
    const response = await axios.delete(
      `http://127.0.0.1:5000/api/delete_default_application/${deleteId.value}`
    );
    // 如果请求成功，并且状态码为200，则认为删除成功
    if (response.status === 200) {
      deleteLoading.value = false;
      deleteError.value = "删除成功"; // 设置删除成功的消息
    } else {
      // 如果状态码不是200，但请求仍然成功，可以根据返回的状态码或消息进行处理
      deleteLoading.value = false;
      deleteError.value = response.data.message || "删除操作失败";
    }
  } catch (err) {
    console.error(err);
    deleteLoading.value = false;
    if (err.response && err.response.status === 404) {
      deleteError.value = "无对应记录";
    } else {
      deleteError.value = "删除记录出错";
    }
  }
};
const fetchPendingRebirths = async () => {
  loading.value = true;
  error.value = null;
  try {
    const response = await axios.get("/default_rebirths/review");
    pendingRebirths.value = response.data;
  } catch (err) {
    console.error(err);
    error.value = "加载待审核申请失败";
  } finally {
    loading.value = false;
  }
};

const submitReview = async (id, auditStatus) => {
  try {
    const response = await axios.post(`/default_rebirths/review`, {
      id: id,
      audit_status: auditStatus,
    });
    if (response.status === 200) {
      fetchPendingRebirths(); // 重新加载待审核申请
    }
  } catch (err) {
    console.error(err);
    error.value = "审核提交失败";
  }
};
onMounted(() => {
  fetchRegionData();
  fetchIndustryData();
}); // 在组件挂载时获取数据

// 定义其他数据
const activities = [
  {
    content: "收藏商品",
    description: "xxx收藏了你的商品，就是不买",
    timestamp: "30分钟前",
    color: "#00bcd4",
  },
  {
    content: "用户评价",
    description: "xxx给了某某商品一个差评，吐血啊",
    timestamp: "55分钟前",
    color: "#1ABC9C",
  },
  {
    content: "订单提交",
    description: "xxx提交了订单，快去收钱吧",
    timestamp: "1小时前",
    color: "#3f51b5",
  },
  {
    content: "退款申请",
    description: "xxx申请了仅退款，又要亏钱了",
    timestamp: "15小时前",
    color: "#f44336",
  },
  {
    content: "商品上架",
    description: "运营专员瞒着你上架了一辆飞机",
    timestamp: "1天前",
    color: "#009688",
  },
];

const ranks = [
  {
    title: "手机",
    value: 10000,
    percent: 80,
    color: "#f25e43",
  },
  {
    title: "电脑",
    value: 8000,
    percent: 70,
    color: "#00bcd4",
  },
  {
    title: "相机",
    value: 6000,
    percent: 60,
    color: "#64d572",
  },
  {
    title: "衣服",
    value: 5000,
    percent: 55,
    color: "#e9a745",
  },
  {
    title: "书籍",
    value: 4000,
    percent: 50,
    color: "#009688",
  },
];
</script>

<style>
.card-body {
  display: flex;
  align-items: center;
  height: 100px;
  padding: 0;
}
.chart {
  width: 100%;
  height: 400px;
}
</style>
<style scoped>
.card-content {
  flex: 1;
  text-align: center;
  font-size: 14px;
  color: #999;
  padding: 0 20px;
}

.card-num {
  font-size: 30px;
}

.card-icon {
  font-size: 50px;
  width: 100px;
  height: 100px;
  text-align: center;
  line-height: 100px;
  color: #fff;
}
.search-section {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.search-section el-input {
  margin-right: 10px;
}
.search-section {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.search-section el-input {
  margin-right: 10px;
}

.delete-section {
  display: flex;
  align-items: center;
  margin-top: 20px;
}

.delete-section el-input {
  margin-right: 10px;
}

.error {
  color: red;
}
.error {
  color: red;
}
.bg1 {
  background: #2d8cf0;
}

.bg2 {
  background: #64d572;
}

.bg3 {
  background: #f25e43;
}

.bg4 {
  background: #e9a745;
}

.color1 {
  color: #2d8cf0;
}

.color2 {
  color: #64d572;
}

.color3 {
  color: #f25e43;
}

.color4 {
  color: #e9a745;
}

.chart {
  width: 100%;
  height: 400px;
}

.card-header {
  padding-left: 10px;
  margin-bottom: 20px;
}

.card-header-title {
  font-size: 18px;
  font-weight: bold;
  margin-bottom: 5px;
}

.card-header-desc {
  font-size: 14px;
  color: #999;
}

.timeline-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 16px;
  color: #000;
}

.timeline-time,
.timeline-desc {
  font-size: 12px;
  color: #787878;
}

.rank-item {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.rank-item-avatar {
  width: 40px;
  height: 40px;
  border-radius: 50%;
  background: #f2f2f2;
  text-align: center;
  line-height: 40px;
  margin-right: 10px;
}

.rank-item-content {
  flex: 1;
}

.rank-item-top {
  display: flex;
  justify-content: space-between;
  align-items: center;
  color: #343434;
  margin-bottom: 10px;
}

.rank-item-desc {
  font-size: 14px;
  color: #999;
}
.map-chart {
  width: 100%;
  height: 350px;
}
</style>
