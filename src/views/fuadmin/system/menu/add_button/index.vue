<template>
  <BasicDrawer v-bind="$attrs" @register="registerDrawerMenu" :title="getTitle" width="50%">
    <BasicTable @register="registerTable">
      <template #tableTitle>
        <a-button type="primary" @click="handleCreate"> {{ t('common.addText') }} </a-button>
      </template>
      <template #bodyCell="{ column, record }">
        <template v-if="column.key === 'action'">
          <TableAction
            :actions="[
              {
                type: 'button',
                icon: 'clarity:note-edit-line',
                color: 'primary',
                onClick: handleEdit.bind(null, record),
              },
              {
                icon: 'ant-design:delete-outlined',
                type: 'button',
                color: 'error',
                placement: 'left',
                popConfirm: {
                  title: t('common.delHintText'),
                  confirm: handleDelete.bind(null, record.id),
                },
              },
            ]"
          />
        </template>
      </template>
    </BasicTable>
    <MenuButtonDrawer @register="registerDrawer" @success="handleSuccess" />
  </BasicDrawer>
</template>
<script lang="ts">
  import { defineComponent, ref, unref } from 'vue';

  import { BasicTable, useTable, TableAction } from '/@/components/Table';

  import { BasicDrawer, useDrawer, useDrawerInner } from '/@/components/Drawer';
  import { deleteItem, getList } from './menu_button.api';
  import { columns } from './menu_button.data';
  import MenuButtonDrawer from '/@/views/fuadmin/system/menu/add_button/MenuButtonDrawer.vue';
  import { useI18n } from '/@/hooks/web/useI18n';

  export default defineComponent({
    name: 'AddMenuButton',
    components: { BasicTable, MenuButtonDrawer, BasicDrawer, TableAction },
    setup() {
      const { t } = useI18n();

      const [registerDrawer, { openDrawer }] = useDrawer();
      const menuId = ref(0);
      const [registerDrawerMenu] = useDrawerInner(async (data) => {
        menuId.value = data.id;
        await reload();
      });
      const getTitle = '??????????????????';

      const [registerTable, { reload }] = useTable({
        api: getList,
        columns,
        showTableSetting: true,
        bordered: true,
        immediate: false,
        showIndexColumn: false,
        searchInfo: {
          menu_id: menuId,
        },
        actionColumn: {
          width: 50,
          title: t('common.operationText'),
          dataIndex: 'action',
          fixed: 'right',
        },
      });

      function handleCreate() {
        openDrawer(true, {
          isUpdate: false,
          menuId: unref(menuId),
        });
      }

      function handleEdit(record: Recordable) {
        openDrawer(true, {
          record,
          isUpdate: true,
          menuId: unref(menuId),
        });
      }

      async function handleDelete(id: number) {
        await deleteItem(id);
        await reload();
      }

      function handleSuccess() {
        reload();
      }

      return {
        registerTable,
        registerDrawer,
        handleCreate,
        handleEdit,
        handleDelete,
        handleSuccess,
        registerDrawerMenu,
        getTitle,
        t,
      };
    },
  });
</script>
