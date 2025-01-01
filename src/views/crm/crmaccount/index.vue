<template>
  <ContentWrap>
    <!-- 搜索工作栏 -->
    <el-form
      class="-mb-15px"
      :model="queryParams"
      ref="queryFormRef"
      :inline="true"
      label-width="100px"
    >
      <el-form-item label="归属年月" prop="name">
            <el-date-picker
              v-model="queryParams.createTime"
              value-format="YYYY/MM/DD"
              format="YYYY/MM/DD"
              type="datetimerange"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              style="width: 220px !important;"
            />
      </el-form-item>
      <el-form-item label="交易日期" prop="mobile">
        <el-date-picker
              v-model="queryParams.jyrq"
              value-format="YYYY/MM/DD"
              format="YYYY/MM/DD"
              type="datetimerange"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              style="width: 220px !important;"
            />
      </el-form-item>
      <el-form-item label="交易方向" prop="telephone">
        <el-select v-model="queryParams.source" clearable placeholder="请选择交易方向" class="!w-240px">
          <el-option
              v-for="dict in getIntDictOptions(DICT_TYPE.TRADE_DIRECTION)"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value"
            />
        </el-select>
      </el-form-item>
      <el-form-item label="款项内容" prop="dealStatus">
        <el-input
          v-model="queryParams.telephone"
          placeholder="请输入款项内容"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="收款公司" prop="level">
        <el-input
          v-model="queryParams.telephone"
          placeholder="请输入收款公司"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="收款经手人" prop="industry">
        <el-input
          v-model="queryParams.telephone"
          placeholder="请输入收款经手人"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="付款公司" prop="industry">
        <el-input
          v-model="queryParams.telephone"
          placeholder="请输入付款公司"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="付款经手人" prop="industry">
        <el-input
          v-model="queryParams.telephone"
          placeholder="请输入付款经手人"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="票号" prop="industry">
        <el-input
          v-model="queryParams.telephone"
          placeholder="请输入票号"
          clearable
          @keyup.enter="handleQuery"
          class="!w-240px"
        />
      </el-form-item>
      <el-form-item label="到期兑换时间" prop="mobile">
        <el-date-picker
              v-model="queryParams.jyrq"
              value-format="YYYY/MM/DD"
              format="YYYY/MM/DD"
              type="datetimerange"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              style="width: 220px !important;"
            />
      </el-form-item>
      <el-form-item label="交易方式" prop="source">
        <el-select v-model="queryParams.source" clearable placeholder="请选择交易方式" class="!w-240px">
          <el-option
              v-for="dict in getIntDictOptions(DICT_TYPE.TRADE_TYPE)"
              :key="dict.value"
              :label="dict.label"
              :value="dict.value"
            />
        </el-select>
      </el-form-item>
      
      <el-form-item>
        <el-button @click="handleQuery"><Icon icon="ep:search" class="mr-5px" /> 搜索</el-button>
        <el-button @click="resetQuery"><Icon icon="ep:refresh" class="mr-5px" /> 重置</el-button>
      </el-form-item>
    </el-form>
  </ContentWrap>

  <!-- 列表 -->
  <ContentWrap>
    <el-table v-loading="loading" :data="list" :stripe="true" :show-overflow-tooltip="true">
      <el-table-column label="归属年月" align="center" prop="id" />
      <el-table-column label="交易日期" align="center" prop="name" width="200" />
      <el-table-column label="交易方向" align="center" prop="name" width="120" />
      <el-table-column label="款项内容" align="center" prop="name" width="120" />
      <el-table-column label="交易金额(元)" align="center" prop="name" width="120" />
      <el-table-column label="付款公司" align="center" prop="name" width="120" />
      <el-table-column label="付款经手人" align="center" prop="name" width="120" />
      <el-table-column label="收款公司" align="center" prop="name" width="120" />
      <el-table-column label="收款经手人" align="center" prop="name" width="120" />
      <el-table-column label="交易方式" align="center" prop="name" width="120" />
      <el-table-column label="票号" align="center" prop="name" width="120" />
      <el-table-column label="到期兑换时间" align="center" prop="name" width="120" />
      <el-table-column label="操作" align="center" fixed="right" width="150">
        <template #default="scope">
          <el-button
            link
            type="success"
            @click="handleReceive(scope.row.id)"
          >
            编辑
          </el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页 -->
    <Pagination
      :total="total"
      v-model:page="queryParams.pageNo"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />
  </ContentWrap>

</template>

<script setup lang="ts">
import { dateFormatter } from '@/utils/formatTime'
import download from '@/utils/download'
import { CustomerApi, CustomerVO } from '@/api/crm/crmcustomer'
import { DICT_TYPE, getIntDictOptions } from '@/utils/dict'


/** 客户 列表 */
defineOptions({ name: 'CrmCustomer' })

const message = useMessage() // 消息弹窗
const { t } = useI18n() // 国际化

const loading = ref(true) // 列表的加载中
const list = ref<CustomerVO[]>([]) // 列表的数据
const total = ref(0) // 列表的总页数
const queryParams = reactive({
  pageNo: 1,
  pageSize: 10,
  jyrq: undefined,
  name: undefined,
  mobile: undefined,
  telephone: undefined,
  dealStatus: undefined,
  level: undefined,
  industry: undefined,
  tags: undefined,
  source: undefined,
  remark: undefined,
  userId: undefined,
  weixin: undefined,
  qq: undefined,
  type: 'open'
})
const queryFormRef = ref() // 搜索的表单
const exportLoading = ref(false) // 导出的加载中

/** 查询列表 */
const getList = async () => {
  loading.value = true
  try {
    const data = await CustomerApi.getCustomerPage(queryParams)
    list.value = data.list
    list.value.map((value) => {
        if(value.tags != ''){
          value.tags = value.tags.split(',')
        }
      })
    total.value = data.total
  } finally {
    loading.value = false
  }
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



const handleReceive = async (id: number) => {
  try {
    // 删除的二次确认
    await message.confirm('确认要领取？')
    // 发起删除
    await CustomerApi.receCustomer(id)
    message.success('领取成功')
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
    const data = await CustomerApi.exportCustomer(queryParams)
    download.excel(data, '客户.xls')
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