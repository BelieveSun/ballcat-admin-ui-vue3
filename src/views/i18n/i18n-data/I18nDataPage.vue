<template>
  <i18n-data-page-search :loading="tableRef?.loading" @search="searchTable" />
  <pro-table
    ref="tableRef"
    row-key="id"
    :header-title="t('i18n.i18nData.text')"
    :request="tableRequest"
    :columns="columns"
    :scroll="{ x: 1100 }"
  >
    <template #toolBarRender>
      <export-button @click="handleExport(searchParams)" />
      <import-button @click="handleImport" />
      <new-button @click="handleNew" />
    </template>
    <template #bodyCell="{ column, record }">
      <template v-if="column.key === 'operate'">
        <operation-group>
          <a v-if="hasPermission('i18n:i18n-data:edit')" @click="handleEdit(record)">
            {{ t('action.edit') }}
          </a>
          <delete-text-button
            v-if="hasPermission('i18n:i18n-data:del')"
            @confirm="handleDelete(record)"
          />
        </operation-group>
      </template>
    </template>
  </pro-table>

  <i18n-data-create-modal ref="i18nDataCreateModalRef" @submit-success="reloadTable" />
  <i18n-data-update-modal ref="i18nDataUpdateModalRef" @submit-success="reloadTable" />
  <i18n-data-import-modal ref="i18nDataImportModalRef" @submit-success="reloadTable" />
</template>

<script setup lang="ts">
import ProTable from '#/table'
import type { ProTableInstanceExpose, ProColumns, TableRequest } from '#/table'
import { pageI18nData, deleteI18nData, exportI18nDataExcel } from '@/api/i18n/i18n-data'
import type { I18nDataPageVO, I18nDataQO } from '@/api/i18n/types'
import I18nDataPageSearch from './I18nDataPageSearch.vue'
import I18nDataCreateModal from './I18nDataCreateModal.vue'
import I18nDataUpdateModal from './I18nDataUpdateModal.vue'
import I18nDataImportModal from './I18nDataImportModal.vue'
import { mergePageParam } from '@/utils/page-utils'
import { useAuthorize } from '@/hooks/permission'
import { doRequest } from '@/utils/axios/request'
import { remoteFileDownload } from '@/utils/file-utils'
import { NewButton, ExportButton, ImportButton, DeleteTextButton } from '@/components/Button'
import { OperationGroup } from '@/components/Operation'
import { useAdminI18n } from '@/hooks/i18n'

const { t } = useAdminI18n()

// 鉴权方法
const { hasPermission } = useAuthorize()

const tableRef = ref<ProTableInstanceExpose>()

const i18nDataCreateModalRef = ref()
const i18nDataUpdateModalRef = ref()
const i18nDataImportModalRef = ref()

// 远程搜索参数
let searchParams: I18nDataQO = {}

const tableRequest: TableRequest = async (params, sorter, filter) => {
  const pageParam = mergePageParam(params, sorter, filter)
  return pageI18nData({ ...pageParam, ...searchParams })
}

/* 刷新表格 */
const reloadTable = (resetPageIndex?: boolean) => {
  tableRef.value?.actionRef?.reload(resetPageIndex)
}

/** 查询表格 */
const searchTable = (params: I18nDataQO) => {
  searchParams = params
  reloadTable(true)
}

/* 新建数据 */
const handleNew = () => {
  i18nDataCreateModalRef.value.open()
}

/* 编辑数据 */
const handleEdit = (record: I18nDataPageVO) => {
  i18nDataUpdateModalRef.value.open(record)
}

/* 删除数据 */
const handleDelete = (record: I18nDataPageVO) => {
  doRequest(deleteI18nData(record.code, record.languageTag), {
    successMessage: t('message.removeSuccess'),
    onSuccess: () => reloadTable()
  })
}

/** 导出数据 */
const handleExport = (params: I18nDataQO) => {
  exportI18nDataExcel(params).then(response => {
    remoteFileDownload(response)
  })
}

/** 导入数据 */
const handleImport = () => {
  i18nDataImportModalRef.value.open()
}

const columns = computed<ProColumns[]>(
  () =>
    [
      {
        title: 'ID',
        dataIndex: 'id'
      },
      {
        title: t('i18n.i18nData.languageTag.text'),
        dataIndex: 'languageTag'
      },
      {
        title: t('i18n.i18nData.code.text'),
        dataIndex: 'code'
      },
      {
        title: t('i18n.i18nData.message.text'),
        dataIndex: 'message'
      },
      {
        title: t('common.remarks'),
        dataIndex: 'remarks'
      },
      {
        title: t('common.createTime'),
        dataIndex: 'createTime',
        width: '150px',
        sorter: true
      },
      {
        key: 'operate',
        title: t('common.operation'),
        align: 'center',
        width: '150px'
      }
    ] as ProColumns[]
)
</script>

<script lang="ts">
export default {
  name: 'I18nDataPage'
}
</script>
