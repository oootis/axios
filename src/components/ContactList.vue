<template>
  <div>
    <van-contact-list
     :list="list"
     @add="onAdd"
     @edit="onEdit"
   />
    <van-popup v-model="showEdit" position="bottom">
      <van-contact-edit
       :contact-info="editingContact"
       :is-edit="isEdit"
       @save="onSave"
       @delete="onDelete"
     />
    </van-popup>
  </div>
</template>

<script>
import axios from 'axios'
import {ContactList, Toast, ContactEdit, Popup} from 'vant'
//因为Toast组件不用在模板中渲染，所以不用注册
export default {
    data(){
        return{
            list: [],
            instance: null,
            showEdit: false,
            editingContact: {},//正在编辑的联系人数据
            isEdit: false//新建或编辑
        }
    },
    components: {
        [ContactList.name]: ContactList,
        [ContactEdit.name]: ContactEdit,
        [Popup.name]: Popup
    },
    created(){
        this.instance = axios.create({
            baseURL: 'http://localhost:9000/api',
            timeout: 1000
        });
        this.getList();

    },
    methods: {
    // 添加联系人
    onAdd() {
      this.isEdit = false;
      this.showEdit = true;
    },

    // 编辑联系人
    onEdit(info) {
     this.isEdit = true;      
     this.showEdit = true;
     this.editingContact = info;
    },
    //获取联系人
    getList(){
         this.instance.get('/contactList')
        .then(res=>{
            this.list = res.data.data;
            console.log(res);
        }).catch(err=>{
            Toast('请求失败');
            console.log(err);
        })
    },
    // 保存联系人
    onSave(info) {
      if (this.isEdit){
          this.instance.put('/contact/edit', info)
          .then(res=>{
              if(res.data.code === 200){
                   Toast('sucess');
                   this.showEdit = false;
                   this.getList(); 
              }
          }).catch(()=>{
              Toast('fail')
          })    
      }else{
          this.instance.post('/contact/new/json', info)
          .then(res=>{
              if(res.data.code === 200){
                   Toast('sucess');
                   this.showEdit = false;
                   this.getList(); 
              }
          }).catch(()=>{
              Toast('fail')
          })       
    }
    },
    // 删除联系人
    onDelete(info){
        this.instance.delete('/contact', {
            params:{
                id:info.id
            }
        }).then(res=>{
            if(res.data.code === 200){
                Toast('sucess');
                this.showEdit = false;
                this.getList(); 
            }   
        }).catch(()=>{
            Toast('fail')
        })
    }
    }
    }
</script>

<style>

</style>