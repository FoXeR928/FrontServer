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
                <q-btn v-on:click="show = false, show_all = true, show_get = false, show_give = false, show_search=false, data_start()" align="center" class="btn-fixed-width" color="primary" icon="home" />
                <q-btn v-on:click="show = true, show_all = false, show_get = false, show_give = false, show_search=false" align="center" class="btn-fixed-width" color="primary" label="Добавить флешку" />
                <q-btn v-on:click="show_give = true, show_all = false, show = false, show_get = false, show_search=false" align="center" class="btn-fixed-width" color="primary" label="Выдать флешку" />
                <q-btn v-on:click="show_get = true, show_all = false, show = false, show_give = false, show_search=false" align="center" class="btn-fixed-width" color="primary" label="Вернуть флешку" />
                <q-btn v-on:click="show = false, show_all = false, show_get = false, show_give = false, show_search=true" align="center" class="btn-fixed-width" color="primary" label="Найти флешку" />
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
              
            </form>
            <form v-if="show" enctype="multipart/form-data" id="uploadForm" class="q-gutter-md column items-start flex flex-center" style="margin: 1px 0" method="POST">
              <template>
                <div class="q-pa-md">
                  <div class="q-gutter-sm row items-start">
                    <q-uploader
                      url="http://localhost:800/upload_file"
                      style="max-width: 300px"
                      name='file_txt'
                    />
                  </div>
                </div>
              </template>
              <template>
                <div class="q-pa-md">
                  <div class="q-gutter-sm row items-start">
                    <q-uploader
                      url="http://localhost:800/upload_file"
                      style="max-width: 300px"
                      name='file_reg'
                    />
                  </div>
                </div>
              </template>
              <q-btn align="center" @click="add_flask(file_txt, file_reg)" class="btn-fixed-width" color="primary" label="Добавить" />
            </form>
            <form v-if="show_give" method="PUT">
              <div class="q-gutter-md">
                <q-input v-model="device_id" label="id флешки" />
              </div>
              <div class="q-gutter-md">
                <q-input v-model="fio" label="ФИО получателя" />
              </div>
              <div class="q-gutter-md">
                <q-input type="number" v-model="tabnum" label="Табельный номер получателя" />
              </div>
              <div class="q-gutter-md">
                <q-input v-model="department" label="Депортамет получателя" />
              </div>
              <div class="flex flex-center" style="margin: 20px">
                <q-btn align="center" @click="give_flask(device_id, fio, tabnum, department)" class="btn-fixed-width" color="primary" label="Выдать" />
              </div>
            </form>
            <form v-if="show_get" method="DELETE">
              <div class="q-gutter-md">
                <q-input v-model="device_id" label="id флешки" />
              </div>
              <div class="flex flex-center" style="margin: 20px">
                <q-btn align="center" @click="get_flask(device_id)" class="btn-fixed-width" color="primary" label="Вернуть" />
              </div>
            </form>
            <form action="" method="GET" v-if="show_search">
              <div class="q-gutter-md">
                <q-input v-model="device_id" label="id флешки" />  
              </div>
              <div class="flex space-between" style="justify-content: space-between; margin: 10px 0;">
                <q-btn @click="search_date(device_id)" align="center" class="btn-fixed-width" color="primary" label="Поиск данных флешки" />
                <q-btn @click="search_off()" align="center" class="btn-fixed-width" color="primary" label="Списанные флешки" />
              </div>
            </form>
            <template>
              <div style="padding: 16px 0" class="q-pa-md">
                <q-table
                  :columns="columns"
                  row-key="0"
                  :data="result"
                  :rows-per-page-options="[0]"
                />
              </div>
            </template>
        </section>
    </q-page-container>
  </q-layout>
</template>

<script> 
import axios from 'axios'
export default { 
  data(){
    return{
      device_id: '',
      file_txt: '',
      file_reg: '',
      fio: '',
      tabnum: '',
      department: '',
      result: axios.get("http://localhost:800/all_flask").then(response => (this.result = response.data.Base)),
      result_work: '',
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
        { name: 'device_path', align: 'center', label: 'Файлы txt', field: '1', sortable: true },
        { name: 'device_reg', align: 'center', label: 'Файлы reg', field: '2', sortable: true },
        { name: 'date_in_base', align: 'center', label: 'Дата введения', field: '3', sortable: true },
        { name: 'date_out_base', align: 'center', label: 'Дата выдачи', field: '4', sortable: true },
        { name: 'fio_base', align: 'center', label: 'Сотрудник', field: '5', sortable: true },
        { name: 'tabnum_base', align: 'center', label: 'Табельный номер', field: '6', sortable: true},
        { name: 'department_base', align: 'center', label: 'Департамент', field: '7', sortable: true}],
    }
  },
  methods: {
    add_flask(file_txt, file_reg){
      const date_file_txt = new FormData()
      const date_file_reg = new FormData()
      date_file_txt.append('file', file_txt)
      date_file_reg.append('file', file_reg)
      axios.post("http://localhost:800/upload_file",(date_file_txt, date_file_reg), {headers: {
        'Content-Type': 'multipart/form-data'
      }}).then(response =>(this.$q.notify({
        message: response,
        color: 'primal'
      })))
    },
    data_start(){
      axios.get("http://localhost:800/all_flask").then(response => (this.result = response.data.Base))
    },
    get_flask(device_id){
      axios.delete("http://localhost:800/get_flask?device_id="+device_id,
      ).then(response =>(this.$q.notify({
        message: response,
        color: 'primal'
      })))
    },
    give_flask(device_id, fio, tabnum, department){
      axios.put("http://localhost:800/give_flask?device_id="+device_id+"&fio="+fio+"&tabnum="+tabnum+"&department="+department,{
      }).then(response =>(this.$q.notify({
        message: response,
        color: 'primal'
      })))
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
    }
  }
} 
</script>