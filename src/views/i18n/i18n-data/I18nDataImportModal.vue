<template>
  <a-modal
    :title="title"
    :visible="visible"
    :mask-closable="false"
    :body-style="{ paddingBottom: '8px' }"
    :confirm-loading="submitLoading"
    :width="600"
    @ok="handleSubmit"
    @cancel="handleClose"
  >
    <a-form ref="formRef" :model="formModel" :label-col="labelCol" :wrapper-col="wrapperCol">
      <a-form-item
        name="file"
        :rules="[{ required: true, message: t('message.pleaseSelectFile') }]"
      >
        <a-upload v-model:file-list="fileList" accept=".xls,.xlsx" :before-upload="beforeUpload">
          <a-button>
            <UploadOutlined />
            {{ t('action.selectFile') }}
          </a-button>
          <a href="javascript:" style="margin-left: 12px" @click.stop="downloadTemplate">
            {{ t('import.downloadTemplate') }}
          </a>
        </a-upload>
      </a-form-item>

      <a-form-item name="importMode" :label="t('import.whenDataExisting')">
        <a-radio-group v-model:value="formModel.importMode">
          <a-radio :value="ImportMode.SKIP_EXISTING">
            {{ t('import.skipExisting') }}
          </a-radio>
          <a-radio :value="ImportMode.OVERWRITE_EXISTING">
            {{ t('import.overwriteExisting') }}
          </a-radio>
        </a-radio-group>
      </a-form-item>
    </a-form>
  </a-modal>
</template>

<script setup lang="ts">
import { useModal } from '@/hooks/modal'
import { labelCol, wrapperCol } from '@/hooks/form'
import { downloadI18nDataExcelTemplate, importI18nDataExcel } from '@/api/i18n/i18n-data'
import { useI18n } from 'vue-i18n'
import { remoteFileDownload } from '@/utils/file-utils'
import { ImportMode } from '@/api/types'
import type { UploadFile } from 'ant-design-vue/lib/upload/interface'
import { doRequest } from '@/utils/axios/request'
import type { I18nImportData } from '@/api/i18n/types'

const { t } = useI18n()

const emits = defineEmits<{
  (e: 'submit-success'): void
}>()

const formRef = ref()

const { title, visible, openModal, closeModal } = useModal('导入国际化信息')

const fileList = ref<UploadFile[]>([])

const submitLoading = ref(false)

const formModel = reactive<I18nImportData>({
  file: undefined,
  importMode: ImportMode.SKIP_EXISTING
})

function beforeUpload(file: UploadFile) {
  fileList.value = [file]
  formModel.file = file
  return false
}

/* 下载 excel 模板 */
function downloadTemplate() {
  downloadI18nDataExcelTemplate().then(response => {
    remoteFileDownload(response)
  })
}

/* 表单提交处理 */
const handleSubmit = () => {
  formRef.value.validateFields().then(() => {
    submitLoading.value = true
    doRequest(importI18nDataExcel(toRaw(formModel)), {
      onSuccess: res => {
        closeModal()
        emits('submit-success')
      },
      onFinally: () => {
        submitLoading.value = false
      }
    })
  })
}

/** 弹窗关闭方法 */
const handleClose = () => {
  closeModal()
}

defineExpose({
  open() {
    openModal()
    fileList.value = []
    formModel.importMode = ImportMode.SKIP_EXISTING
    formModel.file = undefined
  }
})
</script>

<script lang="ts">
export default {
  name: 'I18nDataImportModal'
}
</script>

<style scoped></style>
