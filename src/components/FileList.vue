<template>
  <div class="wrapper" :id="'file-' + fileName">
    <div class="left">
      <img
        :src="fileUrl"
        class="image"
      />
    </div>

    <div class="middle" @click="goDir">
      <div :class="{dir: !isFile}">{{ fileName }}</div>
      <div class="size">{{ isFile ? data.sizeLabel : '' }}</div>

      <input
        type="file"
        class="file-input"
        v-if="isFile"
        @change="handleUpdateFile"
      />
    </div>

    <div>
      <slot name="right"></slot>
    </div>
  </div>
</template>

<script lang="ts" setup>
import { PropType, ref } from 'vue'
import { useStore } from 'vuex'
import { useRoute, useRouter } from 'vue-router'
import { getCdn, CDN, updateFileContent } from '@/services'
import type { IFile } from '@/store'
import { getBase64, getFileUrl, isImage } from '@/utils'
import { ElMessage } from 'element-plus'
import { isSuccess } from '@/utils/http'
import { useI18n } from 'vue-i18n'

const props = defineProps({
  data: {
    type: Object as PropType<IFile>,
    default: () => ({}),
  }
})
const { t } = useI18n()
const route = useRoute()
const router = useRouter()
const store = useStore()
const hasError = ref(false)
const imgLoad = ref(true)
const fileName = props.data.name.toLowerCase()
const fileType = props.data.type
const filePath = props.data.path
const cdn1 = getCdn(CDN.Jsdelivr, filePath, false)
const cdn2 = getCdn(CDN.Github, filePath, false)
const markdown = `![](${cdn1})`
const html = `<a href="${cdn1}" target="_blank"><img src="${cdn1}" alt="" /></a>`
const isImg = isImage(fileName)
const isFile = fileType !== 'dir'
const fileUrl = getFileUrl(props.data)

const handleUpdateFile = async function(e: any) {
  const files = e.target.files
  if (files.length <= 0) return

  const file = files[0] as File
  const base64 = await getBase64(file)

  updateFileContent(props.data, {
    content: base64,
    isEncode: false
  }).then(res => {
    if (isSuccess(res.status)) {
      store.dispatch('getDir', route.query.path)
      ElMessage({
        type: 'success',
        message: '更新成功, 由于缓存策略需要次日更新'
      })
    }
  })

  e.target.value = ''
}

function goDir() {
  if (fileType === 'dir') {
    router.replace({
      path: '/app',
      query: {
        path: `/${filePath}`
      }
    })
  }
}
</script>

<style lang="scss" scoped>
.wrapper {
  display: flex;
  cursor: pointer;
  padding: 10px;
  &:active {
    background-color: #f2f2f2;
  }

  .image {
    width: 50px;
    height: 50px;
    object-fit: contain;
    border: 1px solid #f2f2f2;
  }

  .middle {
    position: relative;
    flex: 1;
    margin-left: 7px;
    padding-top: 5px;
    margin-right: 20px;

    .dir {
      margin-top: 5px;
      font-size: 16px;
    }

    .size {
      color: #666;
    }

    div {
      font-size: 14px;
    }

    .file-input {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      opacity: 0;
    }
  }
}
</style>
