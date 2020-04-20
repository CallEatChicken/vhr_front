<template>
  <div>
    <div>
      <div style="position: absolute; padding: 0;">
        <ul>
          <li @click="getAllFolder('')">全部文件</li>
           <li @click="getAllFolder('1')">文档</li>
           <li @click="getAllFolder('0')">视频</li>
        </ul>
      </div>
    </div>
    <Upload
      ref="uploadButton"
      :max-size="fileMaxSize"
      :format="format"
      :on-exceeded-size="handleMaxSize"
      :on-success="handleUploadVideoSuccess"
      :on-format-error="handleFormatError"
      :before-upload="HandleUpload"
      :data="parentItem"
      :action="uploadUrl">
      <Button>Upload</Button>
    </Upload>

   <div>
     <input v-model="foldername"  />
      <i-button type="primary" @click="commit">新建文件夹</i-button>
   </div>
    <div  @click="getAllFolder('')">全部文件</div>
    <div  v-for="(item,index) in folders" :key="index" @click="getCurrentFiles(item,''),getfolders(index)">{{item.title}}</div>
    <div style="padding:100px 100px">
      <template>
      <i-table width="700" border :columns="tableColumns" :data="tableData" stripe></i-table>
        <div style="margin: 10px;overflow: hidden">
          <div style="float: right;">
            <Page :total="flipInfo.totalCount" :pageSize="flipInfo.pageSize" :current="flipInfo.pageNo" @on-change="changePage"></Page>
            <p>共{{flipInfo.totalCount}}条记录</p>
          </div>
        </div>
        </template>
    </div>
  </div>
</template>
<script>
  export default {
    computed: {
      uploadUrl() {
        return 'http://localhost:1367/folder/upload';
      }
      },
    data(){
      return{
        tableColumns:[
          {
            title: '文件名',
            key: 'title',
            width: 200,
            fixed: 'left',
            render: (h, params) => {
              return h(
                    'a',
                    {
                      on: {
                        click: () => {
                          this.getCurrentFiles(params.row)
                        }
                      }
                    },params.row.title
                  )
            }
          },
          {
            title: '创建者',
            key: 'staff_name',
            width: 100,
            fixed: 'left'
          },
          {
            title: '更新时间',
            key: 'createdAt',
            width: 100,
            fixed: 'left'
          },
          {
            title: '大小',
            key: 'size',
            width: 100,
            fixed: 'left'
          },
          {
            title: '创建者',
            key: 'count',
            width: 100,
            fixed: 'left'
          },
          {

            title: '操作',
            key: 'action',
            width: 200,
            align: 'center',
            render: (h, params) => {
              if(params.row.type==true){
              return h('div', [
                h(
                  'Button',
                  {
                    props: {
                      type: 'primary',
                      size: 'small'
                    },
                    style: {},
                    on: {
                      click: () => {
                        this.downloadfile(params)
                      }
                    }
                  },
                  '下载'
                ),
                  h(
                    'Button',
                    {
                      props: {
                        type: 'primary',
                        size: 'small'
                      },
                      style: {},
                      on: {
                        click: () => {
                          this.deletefile(params)
                        }
                      }
                    },
                    '删除'
                  )
              ],
              )
            }
            }
          },
        ],
        tableData:[],
        flipInfo: {
          totalCount: 0,
          pageNo: 1,
          pageSize: 10
        },
        folders:[],
        format: ['txt','pdf','ppt','docx','xlsx','doc','xls','png','jpg','jpeg','gif','mp4','mpg','avi','mov','wmv','mkv','flv'],//上传
        // format: ['.pdf','.pptx','.docx','.xlsx','.xls','.png','.jpg','.jpeg','.mp4','.mpeg','.mpg','.avi','.mov','.wmv','.mkv','.flv'],//上传
        fileMaxSize: 1024 * 20, //文件最大支持100兆
        modal1:false,
        foldername:'',
        parentItem:{}
      }
    },
    mounted(){
      this.getAllFolder('')
    },
    methods:{
      changePage(val){
        debugger
        if(this.folders.length===1&&this.folders[0].kind){
          return this.getTableData(this.parentItem.uuid,this.folders[0].kind,val,this.flipInfo.pageSize)
        }
        if(!this.parentItem.parent_id){
          return this.getTableData('','',val,this.flipInfo.pageSize)
        }else{
          return this.getTableData(this.parentItem.uuid,'',val,this.flipInfo.pageSize)
        }
      },
      getfolders(val){
        this.folders = this.folders.slice(0,val+1)
      },
      downloadfile(val){
      },
      deletefile(val){
        this.nodeRequest("/folder/delete", {
          file:val.row.id
        }).then(resp=> {
          if (resp && resp.data.code == 200) {
            this.tableData = resp.data.data.listInfo
          } else {
          }
        })
      },
      getAllFolder(val){
        this.flipInfo.totalCount=0
        this.flipInfo.pageNo=1
        this.folders=[]
        this.parentItem={}
        if(val==='0'){
          this.folders.push({title:"视频",kind:0})
        }
        if(val==='1'){
          this.folders.push({title:"文档",kind:1})
        }
        this.getTableData('',val,this.flipInfo.pageNo,this.flipInfo.pageSize)
        },
      getTableData(parent_id,val,pageNo,pageSize){
        this.nodeRequest("/folder/query", {
          parent_id:parent_id,
          kind:val,
          pageNo:pageNo,
          pageSize:pageSize
        }).then(resp=> {
          if (resp && resp.data.code == 200) {
            if(resp.data.data.listInfo){
              this.tableData = resp.data.data.listInfo
              this.flipInfo.totalCount  = parseInt(resp.data.data.flipInfo.totalCount)
              this.flipInfo.pageNo = parseInt(resp.data.data.flipInfo.pageNo)
            }
          } else {
            this.$Message.info('网络错误');
          }
        })
      },
        getCurrentFiles(val,b){
          if(!val.uuid){
            this.getAllFolder(val.kind)
          }
          if(val.type === true){
            return console.log("这是文件")
          }
          this.flipInfo.totalCount=0
          this.flipInfo.pageNo=1
          this.parentItem={},
          this.tableData=[],
          this.parentItem=val
          if(this.parentItem.type===false){
            if(b===true){
            this.folders.push(this.parentItem)
            }
            this.getTableData(this.parentItem.uuid,'',this.flipInfo.pageNo,this.flipInfo.pageSize)
          }
        },
      commit(){
        if (this.foldername) {
          let parent_id = ''
          if(!this.parentItem.uuid){
            parent_id='-1'
          }else {
            parent_id = this.parentItem.uuid
          }
          this.nodeRequest("/folder/insert", {
            title: this.foldername,
            parent_id:parent_id,
            path_code:this.parentItem.path_code
          }).then(resp=> {
            if (resp && resp.data.code == 200) {
              if(resp.data.data.token === 201){
                this.$Message.warning({
                  content:this.foldername + '该节点下已存在该文件夹',
                  duration:1
                  });
              }else {
                this.foldername=''
                this.getCurrentFiles(this.parentItem,false)
                this.$Message.success('添加成功');
              }
            } else {
            }
          })
        }
      },
      handleMaxSize (file) {
        this.$Message.warning({
          content:'文件'+ file.name + '太大，上传文件大小不能超过20M.',
          duration:1
        });
      },
      handleFormatError (file) {
        this.$Message.warning(file.name +'文件格式不正确')
      },
      HandleUpload (file) {
        console.log(file.size)
        this.$refs.uploadButton.clearFiles();
      },
      // 点击添加视频--上传视频
      handleUploadVideoSuccess(res, file) {
        this.$Message.success('添加成功');
        this.getCurrentFiles(this.parentItem,false)
      }
    }
  }
</script>
