<!-- web/src/views/admin/Merchants.vue -->
<template>
  <div>
    <div class="header-bar">
      <a-button type="primary" @click="openCreateModal">
        <template #icon><icon-plus /></template>
        新增商户
      </a-button>
    </div>

    <a-table :data="merchants" :loading="loading" :pagination="pagination" @page-change="handlePageChange">
      <template #columns>
        <a-table-column title="ID" data-index="id" :width="80" />
        <a-table-column title="用户名" data-index="username" />
        <a-table-column title="邮箱" data-index="email" />
        <a-table-column title="余额" data-index="balance">
          <template #cell="{ record }">¥{{ record.balance }}</template>
        </a-table-column>
        <a-table-column title="状态" data-index="status" :width="100">
          <template #cell="{ record }">
            <a-tag :color="record.status === 1 ? 'green' : 'red'">
              {{ record.status === 1 ? '正常' : '禁用' }}
            </a-tag>
          </template>
        </a-table-column>
        <a-table-column title="注册时间" data-index="created_at" :width="180" />
        <a-table-column title="操作" :width="120">
          <template #cell="{ record }">
            <a-button type="text" size="small" @click="toggleStatus(record)">
              {{ record.status === 1 ? '禁用' : '启用' }}
            </a-button>
          </template>
        </a-table-column>
      </template>
    </a-table>

    <a-modal
      v-model:visible="createVisible"
      title="新增商户"
      :ok-loading="submitting"
      @ok="handleCreate"
      @cancel="resetCreateForm"
    >
      <a-form :model="createForm" layout="vertical">
        <a-form-item label="用户名" required>
          <a-input v-model="createForm.username" placeholder="4-32个字符" />
        </a-form-item>
        <a-form-item label="初始密码" required>
          <a-input-password v-model="createForm.password" placeholder="6-32个字符" />
        </a-form-item>
        <a-form-item label="邮箱">
          <a-input v-model="createForm.email" placeholder="可选" />
        </a-form-item>
        <a-form-item label="手机号">
          <a-input v-model="createForm.phone" placeholder="可选" />
        </a-form-item>
        <a-alert type="info">商户创建后立即启用，请将初始密码安全交付给商户。</a-alert>
      </a-form>
    </a-modal>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted } from 'vue'
import { Message } from '@arco-design/web-vue'
import { IconPlus } from '@arco-design/web-vue/es/icon'
import { createMerchant, getMerchants, updateMerchantStatus } from '@/api/admin'
import type { Merchant } from '@/api/types'

const loading = ref(false)
const merchants = ref<Merchant[]>([])
const pagination = reactive({
  current: 1,
  pageSize: 20,
  total: 0,
})

const createVisible = ref(false)
const submitting = ref(false)
const createForm = reactive({
  username: '',
  password: '',
  email: '',
  phone: '',
})

const fetchData = async () => {
  loading.value = true
  try {
    const res = await getMerchants({ page: pagination.current, page_size: pagination.pageSize })
    merchants.value = res.data.list
    pagination.total = res.data.total
  } catch (e) {
    // ignore
  } finally {
    loading.value = false
  }
}

const handlePageChange = (page: number) => {
  pagination.current = page
  fetchData()
}

const toggleStatus = async (record: Merchant) => {
  const newStatus = record.status === 1 ? 0 : 1
  try {
    await updateMerchantStatus(record.id, newStatus)
    Message.success('操作成功')
    fetchData()
  } catch (e) {
    // ignore
  }
}

const resetCreateForm = () => {
  createForm.username = ''
  createForm.password = ''
  createForm.email = ''
  createForm.phone = ''
}

const openCreateModal = () => {
  resetCreateForm()
  createVisible.value = true
}

const handleCreate = async () => {
  const username = createForm.username.trim()
  const password = createForm.password

  if (username.length < 4 || username.length > 32) {
    Message.warning('用户名长度必须为4-32个字符')
    return
  }
  if (password.length < 6 || password.length > 32) {
    Message.warning('密码长度必须为6-32个字符')
    return
  }

  submitting.value = true
  try {
    await createMerchant({
      username,
      password,
      email: createForm.email.trim() || undefined,
      phone: createForm.phone.trim() || undefined,
    })
    Message.success('商户创建成功')
    createVisible.value = false
    resetCreateForm()
    fetchData()
  } catch (e) {
    // error handled by interceptor
  } finally {
    submitting.value = false
  }
}

onMounted(() => {
  fetchData()
})
</script>

<style scoped>
.header-bar {
  display: flex;
  justify-content: flex-end;
  margin-bottom: 16px;
}
</style>
