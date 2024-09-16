<template>
    <el-card class="password-reset-container">
        <template #header>
            <div class="header">
                <span>重置密码</span>
            </div>
        </template>
        <el-form :model="passwordForm" ref="passwordFormRef" label-width="120px"
            size="large">
            <el-form-item label="旧密码" required>
                <el-input v-model="passwordForm.oldPassword" type="password"
                    autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="新密码" required :rules="passwordRules">
                <el-input v-model="passwordForm.newPassword" type="password"
                    autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="确认新密码" :rules="confirmPasswordRules">
                <el-input v-model="passwordForm.confirmPassword" type="password"
                    autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="resetPassword">提交</el-button>
            </el-form-item>
        </el-form>
    </el-card>
</template>

<script setup>
import { ref } from 'vue'
import { ElMessage } from 'element-plus'
import { userPasswordResetService } from '@/api/user.js'

import { useTokenStore } from '@/stores/token.js'
const tokenStore = useTokenStore();

import useUserInfoStore from '@/stores/userInfo.js'
const userInfoStore = useUserInfoStore();

import { useRouter } from 'vue-router'
const router = useRouter();

const passwordForm = ref({
    oldPassword: '',
    newPassword: '',
    confirmPassword: ''
})

const passwordRules = ref([{
    required: true,
    message: '请输入新密码',
    trigger: 'blur'
}, {
    min: 5,
    max: 16,
    message: '密码长度必须为 5 到 16 个字符',
    trigger: 'blur'
}]);

const confirmPasswordRules = [{
    validator: (rule, value) => {
        if (value !== passwordForm.value.newPassword) {
            return new Error('两次输入的密码不一致');
        }
        return true;
    },
    trigger: 'blur'
}];

const passwordFormRef = ref(null)

const resetPassword = async () => {
    if (!passwordFormRef.value) return
    const { oldPassword, newPassword, confirmPassword } = passwordForm.value;
    if (newPassword !== confirmPassword) {
        ElMessage.error('两次输入的密码不一致');
        return;
    }
    try {
        const response = await userPasswordResetService(oldPassword, newPassword, confirmPassword);
        if (response.code === 0) {
            ElMessage.success('密码已成功重置，请重新登录');
            // // 清空表单
            // passwordFormRef.value.resetFields();

            // 退出登录
            // 1.清空pinia中存储的token以及个人信息
            tokenStore.removeToken()
            userInfoStore.removeInfo()

            // 2.跳转到登录页面
            router.push('/login')
            ElMessage.success('已退出登录');
        } else {
            ElMessage.error(response.message || '密码重置失败');
        }
    } catch (error) {
        ElMessage.error('修改失败');
        console.log('修改失败:' + error)
    }
}
</script>

<style scoped>
.password-reset-container {
    max-width: 600px;
    margin: 30px auto;
}
</style>