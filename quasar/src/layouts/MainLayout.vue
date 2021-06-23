<template>
  <q-layout view="lHh Lpr lFf">
    <q-header>
      <q-toolbar>
        <q-toolbar-title class="flex flex-center">
          <img style="max-width: 270px; padding: 20px" src="../assets/logo_niti.svg">
        </q-toolbar-title>
      </q-toolbar>
    </q-header>
    <q-page-container class="flex flex-center" style="margin: 50px 0">
      <section class="section-add_flask -section" id='all'>
            <nav class="flex flex-center">
                <q-btn v-on:click="show = false, show_all = true, data_start()" align="center" class="btn-fixed-width" color="primary" label="Найти флешку" />
                <q-btn v-on:click="show = true, show_all = false, data_start()" align="center" class="btn-fixed-width" color="primary" label="Добавить флешку" />
            </nav>
            <form v-if="show_all" action="" method="GET">
              <q-input bottom-slots v-model="fio" label="Поиск по имени">
                <template v-slot:append>
                  <q-icon v-if="fio !== ''" name="close" @click="fio = ''" class="cursor-pointer" />
                </template>
                <template v-slot:after>
                  <q-btn @click="search_device_fio(fio)" round dense flat icon="send" />
                </template>
              </q-input>
              <q-input bottom-slots type="number" v-model="tabnum" label="Поиск по табельному номеру">
                <template v-slot:append>
                  <q-icon v-if="tabnum !== ''" name="close" @click="tabnum = ''" class="cursor-pointer" />
                </template>
                <template v-slot:after>
                  <q-btn @click="search_device_tabnum(tabnum)" round dense flat icon="send" />
                </template>
              </q-input>
              <q-input bottom-slots v-model="device_id" label="Поиск по id">
                <template v-slot:append>
                  <q-icon v-if="device_id !== ''" name="close" @click="device_id = ''" class="cursor-pointer" />
                </template>
                <template v-slot:after>
                  <q-btn @click="search_device_id(device_id)" round dense flat icon="send" />
                </template>
              </q-input>
              <q-btn @click="search_off()" align="center" class="btn-fixed-width" color="primary" label="Списанные флешки" />
            </form>
            <form v-if="show" enctype="multipart/form-data" id="uploadForm" class="q-gutter-md column items-start flex flex-center" style="margin: 1px 0" method="POST">
              <q-file filled v-model="file_txt" label="Файл txt" style="width:300px" counter>
                <template v-slot:prepend>
                  <q-icon name="attach_file" />
                </template>
              </q-file>
              <q-file filled v-model="file_reg" label="Файл txt" style="width:300px" counter>
                <template v-slot:prepend>
                  <q-icon name="attach_file" />
                </template>
              </q-file>
              <q-btn align="center" @click="add_flask(file_txt, file_reg), alert=true, data_start()" class="btn-fixed-width" color="primary" label="Добавить" />
            </form>
              <div style="padding: 16px 0; max-width: 1133px" class="q-pa-md">
                <q-table
                  :columns="columns"
                  row-key="0"
                  :data="result"
                  :rows-per-page-options="[0]"
                  style="height: 400px"
                  virtual-scroll
                  class="my-sticky-virtscroll-table"
                >
                  <template v-slot:header="props">
                    <q-tr :props="props">
                      <q-th auto-width />
                      <q-th
                        v-for="col in props.cols"
                        :key="col.name"
                        :props="props"
                      >
                        {{ col.label }}
                      </q-th>
                    </q-tr>
                  </template>

                  <template v-slot:body="props">
                    <q-tr :props="props">
                      <q-td auto-width>
                        <q-btn size="sm" color="accent" round dense @click="props.expand = !props.expand" :icon="props.expand ? 'remove' : 'add'" />
                      </q-td>
                      <q-td
                        v-for="col in props.cols"
                        :key="col.name"
                        :props="props"
                      >
                        {{ col.value }}
                      </q-td>
                    </q-tr>
                    <q-tr v-show="props.expand" :props="props">
                      <q-td colspan="100%">
                        <div class="text-left">
                          <div>
                            <q-btn align="center"  @click="get_flask(props.row[0]), alert=true, data_start()" class="btn-fixed-width" color="primary" label="Вернуть" />
                            <q-btn v-on:click="show_give = true, device_id=props.row[0]" align="center" style="margin: 0 10px;" class="btn-fixed-width" color="primary" label="Выдать флешку" />
                            <q-btn
                              color="primary"
                              icon-right="archive"
                              label="Скачать файлы"
                              no-caps
                              @click="exportTable"
                            />
                          </div>
                        </div>
                      </q-td>
                    </q-tr>
                  </template>
              </q-table>
              </div>
        </section>
        <q-dialog v-model="alert">
          <q-card>
            <q-card-section>
              <div class="text-h6">Сообщение</div>
            </q-card-section>

            <q-card-section class="q-pt-none">
            {{result_work}}
            </q-card-section>

            <q-card-actions align="right">
              <q-btn flat label="OK" color="primary" v-close-popup />
            </q-card-actions>
          </q-card>
        </q-dialog>
        <q-dialog v-model="show_give">
          <q-card>
            <q-card-section style="width: 500px">
              <div class="text-h6">Выдать фле</div>
            </q-card-section>

            <q-card-section class="q-pt-none" style="width: 500px">
            <form v-if="show_give" method="PUT">
              <div class="q-gutter-md">
                <q-input v-model="fio" label="ФИО получателя" />
              </div>
              <div class="q-gutter-md">
                <q-input type="number" v-model="tabnum" label="Табельный номер получателя" />
              </div>
              <div class="q-gutter-md">
                <q-input v-model="department" label="Депортамет получателя" />
              </div>
            </form>
            </q-card-section>

            <q-card-actions align="center" style="width: 500px">
              <q-btn align="center" @click="give_flask(device_id, fio, tabnum, department), alert=true, data_start()" class="btn-fixed-width" color="primary" label="Выдать" />
            </q-card-actions>
          </q-card>
        </q-dialog>
    </q-page-container>
  </q-layout>
</template>


<style lang="sass">
.my-sticky-virtscroll-table

  .q-table__top,
  .q-table__bottom,
  thead tr:first-child th
    background-color: #fff

  thead tr th
    position: sticky
    z-index: 1
  thead tr:last-child th
    top: 48px
  thead tr:first-child th
    top: 0
</style>
<script> 
import axios from 'axios'
import { exportFile} from 'quasar'
export default { 
  data(){
    return{
      alert: false,
      file_txt: '',
      file_reg: '',
      device_id: '',
      fio: '',
      tabnum: '',
      department: '',
      result: axios.get("http://localhost:800/all_flask").then(response => (this.result = response.data.Base)),
      result_work: 'Проверьте корректность введенных данных',
      show_all: true,
      show_search: false,
      show: false,
      show_give: false,
      show_get: false,
      columns: [
        {
          name: 'device_id',
          label: 'Флешка',
          align: 'center',
          field: '0',
          sortable: true,
          classes: 'bg-grey-2 ellipsis',
          style: 'max-width: 100px',
          headerClasses: 'bg-primary text-white'
        },
        { name: 'date_in_base', align: 'center', label: 'Дата введения', field: '3', sortable: true },
        { name: 'date_out_base', align: 'center', label: 'Дата выдачи', field: '4', sortable: true },
        { name: 'fio_base', align: 'center', label: 'Сотрудник', field: '5', sortable: true },
        { name: 'tabnum_base', align: 'center', label: 'Табельный номер', field: '6', sortable: true},
        { name: 'department_base', align: 'center', label: 'Департамент', field: '7', sortable: true}],
    
    }
  },
  methods: {
    add_flask(file_txt, file_reg){
      let formData = new FormData();
      formData.append('file_txt', this.file_txt);
      formData.append('file_reg', this.file_reg);
      axios.post('http://localhost:800/upload_file', formData, {headers: {
        'Content-Type': 'multipart/form-data'
    }}).then(response => (this.result_work = response[0]))
    },
    data_start(){
      axios.get("http://localhost:800/all_flask").then(response => (this.result = response.data.Base))
    },
    get_flask(device_id){
      axios.delete("http://localhost:800/get_flask?device_id="+device_id,
      ).then(response => (this.result_work = response.data))
    },
    give_flask(device_id, fio, tabnum, department){
      axios.put("http://localhost:800/give_flask?device_id="+device_id+"&fio="+fio+"&tabnum="+tabnum+"&department="+department,{
      }).then(response => (this.result_work = response.data))
    },
    search_device_fio(fio){
      axios.get("http://localhost:800/name_flask?fio="+fio,{
      }).then(response => (this.result = response.data.Flask))
    },
    search_device_tabnum(tabnum){
      axios.get('http://localhost:800/tabnum_flask?tabnum='+tabnum,{
      }).then(response => (this.result = response.data.Flask))
    },
    search_device_id(device_id){
      axios.get("http://localhost:800/id_flask?device_id="+device_id,{
      }).then(response => (this.result = response.data.Flask))
    },
    search_date(device_id){
      axios.get('http://localhost:800/date_flask?device_id='+device_id,{
      }).then(response => (this.result = response.data.Flask))
    },
    search_off(){
      axios.get('http://localhost:800/off_flask',{
      }).then(response => (this.result = response.data.Flask))
    },
    exportTable () {
        

      exportFile(
          'table-export.csv',
          result,
          'text/csv'
        )
    }
  }
} 
</script>