
<!--编写页面静态部分，即view部分-->
<template>
    <div>
      <!--查询表单-->
      <el-form :model="params">
        <el-select v-model="params.siteId" placeholder="请选择站点">
          <el-option
            v-for="item in siteList"
            :key="item.siteId"
            :label="item.siteName"
            :value="item.siteId">
          </el-option>
        </el-select>
        页面别名：<el-input v-model="params.pageAliase"  style="width: 100px"></el-input>
        <el-button type="primary" v-on:click="query"  size="small">查询</el-button>
        <router-link class="mui-tab-item" :to="{path:'/cms/page/add/',query:{
          page: this.params.page,
          siteId: this.params.siteId}}">
          <el-button  type="primary" size="small">新增页面</el-button>
        </router-link>
      </el-form>
      <!--列表-->
      <el-table :data="list" highlight-current-row v-loading="listLoading" style="width: 100%">
        <el-table-column type="index" width="60">
        </el-table-column>
        <el-table-column prop="pageName" label="页面名称" width="300">
        </el-table-column>
        <el-table-column prop="pageAliase" label="别名" width="120">
        </el-table-column>
        <el-table-column prop="pageType" label="页面类型" width="100">
        </el-table-column>
        <el-table-column prop="pageWebPath" label="访问路径" width="200">
        </el-table-column>
        <el-table-column prop="pagePhysicalPath" label="物理路径" width="320">
        </el-table-column>
        <el-table-column prop="pageCreateTime" label="创建时间" width="180" :formatter="formatCreatetime">
        </el-table-column>
        <el-table-column label="操作" width="180">
          <template slot-scope="page">
            <el-button
              size="small"type="text"
              @click="edit(page.row.pageId)">编辑
            </el-button>
            <el-button
              size="small"type="text"
              @click="del(page.row.pageId)">删除
            </el-button>
            <el-button @click="preview(page.row.pageId)" type="text" size="small">页面预览</el-button>
          </template>
        </el-table-column>
      </el-table>
      <!--分页-->
      <el-col :span="24" class="toolbar">
        <el-pagination
          layout="prev, pager, next"
          :page-size="this.params.size"
          v-on:current-change="changePage"
          :total="total"
          :current-page="this.params.page"
          style="float:right;">
        </el-pagination>
      </el-col>
    </div>
</template>

<script>
  /**页面脚本,即model及vm部分*/
  import * as cmsApi from '../api/cms'
  import utilApi from '@/common/utils';
  export default {
    data() {
      return {
        list:[],
        total:50,
        listLoading:false,
        siteList:[], //站点列表
        params:{
          siteId:'',//站点id
          pageAliase:'', //别名
          page:1,//页码
          size:10,//每页显示个数

        }
      }
    },
    created(){
      //存储 请求参数
      /*if(this.$route.query.page){
       this.params.page = Number.parseInt(this.$route.query.page)
       }*/
      this.params.page = Number.parseInt(this.$route.query.page||1);
      this.params.siteId = this.$route.query.siteId||'';
    },
    mounted() {
      //默认查询页面
      this.query()
      //初始化站点列表
      this.siteList = [
        {
          siteId:'5a751fab6abb5044e0d19ea1',
          siteName:'门户主站'
        },
        {
          siteId:'102',
          siteName:'测试站'
        }
      ]
    },
    methods: {
      //分页查询
      changePage(page) {
        this.params.page = page;
        this.query()
      },
      //查询
      query:function () {
        cmsApi.page_list(this.params.page,this.params.size,this.params).then((res)=> {
          console.log(res)
          this.total = res.queryResult.total
          this.list = res.queryResult.list
        })
      },
      formatCreatetime(row, column){
        var createTime = new Date(row.pageCreateTime);
        if (createTime) {
          return utilApi.formatDate(createTime, 'yyyy-MM-dd hh:mm:ss');
        }
      },
      preview:function (pageId) {
        //打开浏览器窗口
        window.open("http://www.xuecheng.com/cms/preview/"+pageId);
      },
      generateHtml (id) {
        // console.log(id)
        this.$router.push({ path: '/cms/page/html/'+id, query:{
            page: this.params.page,
            siteId: this.params.siteId}})
      },
      /*postPage (id) {
        this.$confirm('确认发布该页面吗?', '提示', {
        }).then(() => {
          this.listLoading = true;
          cmsApi.page_postPage(id).then((res) => {
            if(res.success){
              console.log('发布页面id='+id);
              this.listLoading = false;
              this.$message.success('发布成功，请稍后查看结果');
            }else{
              this.$message.error('发布失败');
            }
          });
        }).catch(() => {

        });
      },*/
      edit(pageId){
        this.$router.push({ path: '/cms/page/edit/'+pageId,query:{
            page: this.params.page,
            siteId: this.params.siteId}})
      },
      //删除
      del (index, row) {
        this.$confirm('确认删除该记录吗?', '提示', {
          type: 'warning'
        }).then(() => {
          this.listLoading = true;
          let pageId = row.pageId;
          cmsApi.page_del(pageId).then((res) => {
            this.listLoading = false;
            if(res.success){
              this.$message.success("删除成功")
              this.query();
            }else{
              this.$message.error('删除失败');
            }

          });
        }).catch(() => {

        });
      }
    }
  }
</script>
<style>
  /*编写页面样式*/
</style>
