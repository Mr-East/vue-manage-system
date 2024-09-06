<template>
  <div class="login-bg">
    <div class="login-container">
      <div class="login-header">
        <img class="logo mr10" src="../../assets/img/logo.svg" alt="" />
        <div class="login-title">后台管理系统</div>
      </div>
      <el-form :model="param" :rules="rules" ref="register" size="large">
        <el-form-item
          prop="customer_name"
          label="顾客名"
          label-position="left"
          label-width="80px"
        >
          <el-input v-model="param.customer_name" placeholder="顾客名">
            <template #prefix>
              <el-icon>
                <Message />
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="username" label="用户名" label-width="80px">
          <el-input v-model="param.username" placeholder="用户名">
            <template #prefix>
              <el-icon>
                <User />
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="industry_classification" label="行业名" label-width="80px">
          <el-input v-model="param.industry_classification" placeholder="行业名">
            <template #prefix>
              <el-icon>
                <User />
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="region_classification" label="隶属地" label-width="80px">
          <el-input v-model="param.region_classification" placeholder="隶属地 ">
            <template #prefix>
              <el-icon>
                <User />
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="external_rating" label="外部评级" label-width="80px">
          <el-input v-model="param.external_rating" placeholder="外部评级">
            <template #prefix>
              <el-icon>
                <User />
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item label="组织" label-width="80px">
          <el-input v-model="param.group" placeholder="组织">
            <template #prefix>
              <el-icon>
                <User />
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item label="信用评级" label-width="80px">
          <el-input v-model="param.credit_rating" placeholder="信用评级">
            <template #prefix>
              <el-icon>
                <User />
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-form-item prop="password" label="密码" label-width="80px">
          <el-input
            type="password"
            placeholder="密码"
            v-model="param.password"
            @keyup.enter="submitForm(register)"
          >
            <template #prefix>
              <el-icon>
                <Lock />
              </el-icon>
            </template>
          </el-input>
        </el-form-item>
        <el-button
          class="login-btn"
          type="primary"
          size="large"
          @click="submitForm(register)"
          >注册</el-button
        >
        <p class="login-text">
          已有账号，<el-link type="primary" @click="$router.push('/login')"
            >立即登录</el-link
          >
        </p>
      </el-form>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue';
import { useRouter } from 'vue-router';
import { ElMessage, type FormInstance, type FormRules } from 'element-plus';
import { Register } from '@/types/user';
import service from '@/utils/request';
const router = useRouter();
const param = reactive<Register>({
    customer_name:'',
    username: '',
    password: '',
    industry_classification:'',
    region_classification:'',
    external_rating:'',
    credit_rating:'',
    group:'',
    });

const rules: FormRules = {
    username: [
        {
            required: true,
            message: '请输入用户名',
            trigger: 'blur',
        },
    ],
    password: [{ required: true, message: '请输入密码', trigger: 'blur' }],
    customer_name: [{ required: true, message: '请输入顾客名', trigger: 'blur' }],
    industry_classification: [{ required: true, message: '请输入行业', trigger: 'blur' }],
    region_classification: [{ required: true, message: '请输入隶属地', trigger: 'blur' }],
    external_rating: [{ required: true, message: '请输入外部评级', trigger: 'blur' }],

};
const register = ref<FormInstance>();
    const submitForm = (formEl: FormInstance | undefined) => {
    if (!formEl) return;

    formEl.validate((valid: boolean) => {
        if (valid) {
            service.post('/api/register', param)
            .then((res) => {
              
                if (res.data.code === 200) {
                    ElMessage.success('注册成功，请登录');
                    router.push('/login');
                } else {
                    ElMessage.error(res.message || '注册失败，请稍后再试！');
                }
            })
            .catch((error) => {
                console.error('请求失败：', error);
                // 检查响应存在，并处理400错误
                if (error.message) {
                    // 显示从后端返回的具体错误消息
                    ElMessage.error(error.message || '注册失败，请求错误！');
                } else {
                    // 如果没有响应信息，可能是网络或其他问题
                    ElMessage.error('网络错误或请求未发送成功！');
                }
            });

        } else {
            ElMessage.error('表单验证失败，请检查输入！');
            return false;
        }
    });
};
</script>

<style scoped>
.login-bg {
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  height: 100vh;
  background: url(../../assets/img/login-bg.jpg) center/cover no-repeat;
}

.login-header {
  display: flex;
  align-items: center;
  justify-content: center;
  margin-bottom: 40px;
}

.logo {
  width: 35px;
}

.login-title {
  font-size: 22px;
  color: #333;
  font-weight: bold;
}

.login-container {
  width: 450px;
  border-radius: 5px;
  background: #fff;
  padding: 40px 50px 50px;
  box-sizing: border-box;
}

.login-btn {
  display: block;
  width: 100%;
}

.login-text {
  display: flex;
  align-items: center;
  margin-top: 20px;
  font-size: 14px;
  color: #787878;
}
</style>
