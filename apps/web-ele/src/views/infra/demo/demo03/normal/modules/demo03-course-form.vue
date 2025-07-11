<script lang="ts" setup>
import type { OnActionClickParams } from '#/adapter/vxe-table';
import type { Demo03StudentApi } from '#/api/infra/demo/demo03/normal';

import { nextTick, watch } from 'vue';

import { Plus } from '@vben/icons';

import { ElButton, ElInput } from 'element-plus';

import { useVbenVxeGrid } from '#/adapter/vxe-table';
import { getDemo03CourseListByStudentId } from '#/api/infra/demo/demo03/normal';
import { $t } from '#/locales';

import { useDemo03CourseGridEditColumns } from '../data';

const props = defineProps<{
  studentId?: number; // 学生编号（主表的关联字段）
}>();

/** 表格操作按钮的回调函数 */
function onActionClick({
  code,
  row,
}: OnActionClickParams<Demo03StudentApi.Demo03Course>) {
  switch (code) {
    case 'delete': {
      onDelete(row);
      break;
    }
  }
}

const [Grid, gridApi] = useVbenVxeGrid({
  gridOptions: {
    columns: useDemo03CourseGridEditColumns(onActionClick),
    border: true,
    showOverflow: true,
    autoResize: true,
    keepSource: true,
    rowConfig: {
      keyField: 'id',
    },
    pagerConfig: {
      enabled: false,
    },
    toolbarConfig: {
      enabled: false,
    },
  },
});

/** 添加学生课程 */
const onAdd = async () => {
  await gridApi.grid.insertAt({} as Demo03StudentApi.Demo03Course, -1);
};

/** 删除学生课程 */
const onDelete = async (row: Demo03StudentApi.Demo03Course) => {
  await gridApi.grid.remove(row);
};

/** 提供获取表格数据的方法供父组件调用 */
defineExpose({
  getData: (): Demo03StudentApi.Demo03Course[] => {
    const data = gridApi.grid.getData() as Demo03StudentApi.Demo03Course[];
    const removeRecords =
      gridApi.grid.getRemoveRecords() as Demo03StudentApi.Demo03Course[];
    const insertRecords =
      gridApi.grid.getInsertRecords() as Demo03StudentApi.Demo03Course[];
    return [
      ...data.filter(
        (row) => !removeRecords.some((removed) => removed.id === row.id),
      ),
      ...insertRecords.map((row: any) => ({ ...row, id: undefined })),
    ];
  },
});

/** 监听主表的关联字段的变化，加载对应的子表数据 */
watch(
  () => props.studentId,
  async (val) => {
    if (!val) {
      return;
    }
    await nextTick();
    await gridApi.grid.loadData(
      await getDemo03CourseListByStudentId(props.studentId!),
    );
  },
  { immediate: true },
);
</script>

<template>
  <Grid class="mx-4">
    <template #name="{ row }">
      <ElInput v-model="row.name" />
    </template>
    <template #score="{ row }">
      <ElInput v-model="row.score" />
    </template>
  </Grid>
  <div class="-mt-4 flex justify-center">
    <ElButton
      :icon="Plus"
      type="primary"
      plain
      @click="onAdd"
      v-access:code="['infra:demo03-student:create']"
    >
      {{ $t('ui.actionTitle.create', ['学生课程']) }}
    </ElButton>
  </div>
</template>
