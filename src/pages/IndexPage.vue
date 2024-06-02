<template>
  <q-page class="row q-pt-xl">
    <div class="full-width q-px-xl">
      <div class="q-mb-xl">
        <q-input v-model="tempData.name" label="姓名" />
        <q-input v-model="tempData.age" label="年齡" />
        <q-btn @click ="addItem()" color="primary" class="q-mt-md">新增</q-btn>
      </div>

      <q-table
        flat
        bordered
        ref="tableRef"
        :rows="blockData"
        :columns="(tableConfig as QTableProps['columns'])"
        row-key="id"
        hide-pagination
        separator="cell"
        :rows-per-page-options="[0]"
      >
        <template v-slot:header="props">
          <q-tr :props="props">
            <q-th v-for="col in props.cols" :key="col.name" :props="props">
              {{ col.label }}
            </q-th>
            <q-th></q-th>
          </q-tr>
        </template>

        <template v-slot:body="props">
          <q-tr :props="props">
            <q-td
              v-for="col in props.cols"
              :key="col.name"
              :props="props"
              style="min-width: 120px"
            >
              <div>{{ col.value }}</div>
            </q-td>
            <q-td class="text-right" auto-width v-if="tableButtons.length > 0">
              <q-btn
                @click="handleClickOption(btn, props.row)"
                v-for="(btn, index) in tableButtons"
                :key="index"
                size="sm"
                color="grey-6"
                round
                dense
                :icon="btn.icon"
                class="q-ml-md"
                padding="5px 5px"
              >
                <q-tooltip
                  transition-show="scale"
                  transition-hide="scale"
                  anchor="top middle"
                  self="bottom middle"
                  :offset="[10, 10]"
                >
                  {{ btn.label }}
                </q-tooltip>
              </q-btn>
            </q-td>
          </q-tr>
        </template>

        <template v-slot:no-data="{ icon }">
          <div
            class="full-width row flex-center items-center text-primary q-gutter-sm"
            style="font-size: 18px"
          >
            <q-icon size="2em" :name="icon" />
            <span> 無相關資料 </span>
          </div>
        </template>
      </q-table>
     <q-dialog v-model="prompt" persistent>
      <q-card style="min-width: 350px">
        <q-card-section>
          <div class="text-h6">Your address</div>
        </q-card-section>

        <q-card-section class="q-pt-none">
          <q-input dense v-model="editedData.name" autofocus label="Name"  @keyup.enter="prompt = false" />
          <q-input dense v-model="editedData.age" autofocus label="Age" @keyup.enter="prompt = false" />

        </q-card-section>

        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Cancel" v-close-popup />
          <q-btn flat @click="saveChanges" label="確定更改" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <!-- <q-dialog v-model="deletePrompt" persistent>
      <q-card style="min-width: 350px">
        <q-card-section>
          <div class="text-h6">確定刪除?</div>
        </q-card-section>


        <q-card-actions align="right" class="text-primary">
          <q-btn flat label="Cancel" v-close-popup />
          <q-btn flat @click="deleteComform" label="確定更改" v-close-popup />
        </q-card-actions>
      </q-card>
    </q-dialog> -->
    </div>
  </q-page>
</template>

<script setup lang="ts">
import axios from 'axios';
import { QTableProps } from 'quasar';
import { ref,onMounted } from 'vue';
interface btnType {
  label: string;
  icon: string;
  status: string;
}
const blockData = ref([
  {
    name: 'test',
    age: 25,
  },
]);
const tableConfig = ref([
  {
    label: '姓名',
    name: 'name',
    field: 'name',
    align: 'left',
  },
  {
    label: '年齡',
    name: 'age',
    field: 'age',
    align: 'left',
  },
]);
const tableButtons = ref([
  {
    label: '編輯',
    icon: 'edit',
    status: 'edit',
  },
  {
    label: '刪除',
    icon: 'delete',
    status: 'delete',
  },
]);
const prompt = ref(false); 
const deletePrompt = ref(false); 
const editedData = ref({}); 
const tempData = ref({
  name: '',
  age: '',
});
async function handleClickOption(btn:btnType, data) {
  if (btn.status === 'edit') {
    prompt.value =true
    editedData.value = data
  }
  if (btn.status === 'delete') {
    try {
      const response = await axios.delete(`https://dahua.metcfire.com.tw/api/CRUDTest/${data.id}`)
        if (response.data === true) {
          blockData.value = blockData.value.filter(item => item.id !== data.id); 
        }
    } catch (error) {
       console.log('error',error)
    }
  }
}
const saveChanges = async () => {
    try {
        const postDataTemplate = {
        id: editedData.value.id,
        name: editedData.value.name,
        age: editedData.value.age
      }
        const response = await axios.patch(`https://dahua.metcfire.com.tw/api/CRUDTest`, postDataTemplate);
        prompt.value = false;
    } catch (error) {
     console.log('error',erro)
    }
};
async function addItem(btn:btnType,data){
  try {
    let postDataTemplate = {
          id: '111',
          name: tempData.value.name,
          age: tempData.value.age
        }
    const response = await axios.post('https://dahua.metcfire.com.tw/api/CRUDTest',postDataTemplate)
    if(response.data === true) blockData.value.push(postDataTemplate) 
  } catch (error) {
    console.error('error:', error)
  }  
}
onMounted(async () => {
    const response =  await axios.get('https://dahua.metcfire.com.tw/api/CRUDTest/a')
    blockData.value =response.data

})

</script>

<style lang="scss" scoped>
.q-table th {
  font-size: 20px;
  font-weight: bold;
}

.q-table tbody td {
  font-size: 18px;
}
</style>
