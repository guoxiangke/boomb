<template>
  <el-dropdown placement="bottom-end" :hide-on-click="false">
    <span class="sorter">
      {{ sortType === SortType.FileSize ? t('fileSize') : t('fileName') }}
      <i :class="isUp ? 'el-icon-top' : 'el-icon-bottom'"></i>
    </span>

    <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item
          :icon="Check"
          class="item"
          :class="{active: isUp}"
          @click="isUp = true"
        >
          {{ t('asc' )}}
        </el-dropdown-item>
        <el-dropdown-item
          :icon="Check"
          class="item"
          :class="{active: !isUp}"
          @click="isUp = false"
        >
          {{ t('desc' )}}
        </el-dropdown-item>

        <el-dropdown-item
          divided
          :icon="Check"
          class="item"
          @click="sortType = SortType.FileSize"
          :class="{active: sortType === SortType.FileSize}"
        >
          {{ t('fileSize' )}}
        </el-dropdown-item>
        <el-dropdown-item
          :icon="Check"
          class="item"
          @click="sortType = SortType.FileName"
          :class="{active: sortType === SortType.FileName}"
        >
          {{ t('fileName' )}}
        </el-dropdown-item>
      </el-dropdown-menu>
    </template>
  </el-dropdown>
</template>

<script lang="ts" setup>
import { computed, ref, watch } from 'vue'
import { useStore } from 'vuex'
import { useRoute } from 'vue-router'
import type { IFile } from '@/store'
import { useI18n } from 'vue-i18n'
import { getCharCode } from '@/utils'
import { Check } from '@element-plus/icons-vue'

enum SortType {
  FileSize = 1, // 文件大小
  FileName // 文件名
}

const { t } = useI18n()
const store = useStore()
const route = useRoute()
const isUp = ref(true)
const dir = computed<IFile[]>(() => store.getters.getDir(route))

const sortType = ref(SortType.FileSize)

watch([sortType, isUp], () => {
  let sortDir: IFile[] = []

  switch (sortType.value) {
    case SortType.FileSize:
      sortDir = dir.value.sort((a: IFile, b: IFile) => a.size - b.size)
      break

    case SortType.FileName:
      sortDir = dir.value.sort((a: IFile, b: IFile) => {
        const aCode = getCharCode(a.name)
        const bCode = getCharCode(b.name)
        return aCode - bCode
      })
      break
  }

  // Down sort
  if (!isUp.value) {
    sortDir = sortDir.reverse()
  }

  store.commit('saveDir', {
    data: sortDir,
    path: route.query.path
  })
})
</script>

<style lang="scss" scoped>
.sorter {
  padding: 7px 12px;
  margin-left: 30px;;
  display: flex;
  align-items: center;
  cursor: pointer;
  transition: all .1s linear;
  color: #000;
  font-weight: bold;

  &:hover {
    background: #f2f2f2;
  }
}

.item {

  ::deep(.el-icon-check) {
    opacity: 0 !important;
  }
}

.active {
  color: #2980ff;
  font-weight: bold;

  ::deep(i.el-icon-check) {
    opacity: 1 !important;
  }
}
</style>
