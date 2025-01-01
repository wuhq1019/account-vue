<template>
  <ContentWrap>
    <el-menu
    :default-active="activeIndex"
    style="margin-bottom:10px"
    mode="horizontal"
    @select="handleSelect"
  >
      <el-menu-item index="my">月度实收统计</el-menu-item>
      <el-menu-item index="sub">月度实付统计</el-menu-item>
    </el-menu>
    <!-- 搜索工作栏 -->
    <el-form
      class="-mb-15px"
      :model="queryParams"
      ref="queryFormRef"
      :inline="true"
      label-width="68px"
    >
    <el-form-item label="归属年月" prop="name">
            <el-date-picker
              v-model="queryParams.gsny"
              value-format="YYYY/MM/DD"
              format="YYYY/MM/DD"
              type="datetimerange"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              style="width: 220px !important;"
            />
      </el-form-item>
      <el-form-item label="付款公司" prop="name">
        <el-input
          v-model="queryParams.name"
          placeholder="请输入付款公司"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
     
      <el-form-item>
        <el-button @click="handleQuery"><Icon icon="ep:search" class="mr-5px" /> 搜索</el-button>
        <el-button @click="resetQuery"><Icon icon="ep:refresh" class="mr-5px" /> 重置</el-button>
        <!-- <el-button
          type="primary"
          plain
          @click="openForm('create')"
          v-hasPermi="['crm:customer-contacts:create']"
        >
          <Icon icon="ep:delete" class="mr-5px" /> 回收站
        </el-button> -->
      </el-form-item>
    </el-form>
  </ContentWrap>

  <!-- 列表 -->
  <ContentWrap>
    <el-table v-loading="loading" :data="list" :stripe="true" :show-overflow-tooltip="true">
      <el-table-column label="归属年月" align="center" prop="customerName" width=""  />
      <el-table-column label="付款公司" align="center" prop="name" width="400"  />
      <el-table-column label="现金实收金额(元)" align="center" prop="mobile" width="" />
      <el-table-column label="电子承兑金额(元)" align="center" prop="telephone" width="" />
      <el-table-column label="对私实收金额(元)" align="center" prop="wechat" width="" />
      <el-table-column label="实收总金额(元)" align="center" prop="email" width="" />
     
    </el-table>
    <!-- 分页 -->
    <Pagination
      :total="total"
      v-model:page="queryParams.pageNo"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />
  </ContentWrap>

  <!-- 表单弹窗：添加/修改 -->
  <CustomerContactsForm ref="formRef" @success="getList" />
</template>

<script setup lang="ts">
import { dateFormatter } from '@/utils/formatTime'
import download from '@/utils/download'
import { CustomerContactsApi, CustomerContactsVO } from '@/api/crm/crmcustomercontacts'
import CustomerContactsForm from './CustomerContactsForm.vue'

/** 联系人 列表 */
defineOptions({ name: 'CrmCustomerContacts' })

const message = useMessage() // 消息弹窗
const { t } = useI18n() // 国际化

const loading = ref(true) // 列表的加载中
const list = ref<CustomerContactsVO[]>([]) // 列表的数据
const total = ref(0) // 列表的总页数
const queryParams = reactive({
  pageNo: 1,
  pageSize: 10,
  gsny:undefined,
  customerName: undefined,
  name: undefined,
  mobile: undefined,
  birthday: undefined,
  telephone: undefined,
  wechat: undefined,
  email: undefined,
  decision: undefined,
  post: undefined,
  relation: 'my'
})
const queryFormRef = ref() // 搜索的表单
const exportLoading = ref(false) // 导出的加载中
const activeIndex = ref('my')

/** 查询列表 */
const getList = async () => {
  loading.value = true
  try {
    const data = await CustomerContactsApi.getCustomerContactsPage(queryParams)
    list.value = data.list
    total.value = data.total
  } finally {
    loading.value = false
  }
}

const handleSelect = (key) => {
  queryParams.relation = key
  getList()
}

/** 搜索按钮操作 */
const handleQuery = () => {
  queryParams.pageNo = 1
  getList()
}

/** 重置按钮操作 */
const resetQuery = () => {
  queryFormRef.value.resetFields()
  handleQuery()
}

/** 添加/修改操作 */
const formRef = ref()
const openForm = (type: string, id?: number) => {
  formRef.value.open(type, id)
}

/** 删除按钮操作 */
const handleDelete = async (id: number) => {
  try {
    // 删除的二次确认
    await message.delConfirm()
    // 发起删除
    await CustomerContactsApi.deleteCustomerContacts(id)
    message.success(t('common.delSuccess'))
    // 刷新列表
    await getList()
  } catch {}
}

/** 导出按钮操作 */
const handleExport = async () => {
  try {
    // 导出的二次确认
    await message.exportConfirm()
    // 发起导出
    exportLoading.value = true
    const data = await CustomerContactsApi.exportCustomerContacts(queryParams)
    download.excel(data, '联系人.xls')
  } catch {
  } finally {
    exportLoading.value = false
  }
}

/** 初始化 **/
onMounted(() => {
  getList()
})
</script>