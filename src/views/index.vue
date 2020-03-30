<template>
  <div class="scan">
    <div class="searchBox">
      <el-form ref="form-scan" :model="scanForm" label-width="110px">
        <el-form-item label="单证编号：">
          <el-input v-model="scanForm.dzbh"></el-input>
        </el-form-item>
        <el-form-item label="影像状态：">
          <el-select v-model="scanForm.yxzt" placeholder="请选择">
            <el-option
                v-for="(c, cdex) in yxztArr"
                :key="cdex"
                :label="c.name" 
                :value="c.id">
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="匹配状态：">
          <el-select v-model="scanForm.ppzt" placeholder="请选择">
            <el-option
                v-for="(c, cdex) in ppztArr"
                :key="cdex"
                :label="c.name" 
                :value="c.id">
            </el-option>
          </el-select>
        </el-form-item>
      </el-form>
      <el-form ref="formScan" :model="scanForm" label-width="110px">
        <el-form-item label="扫描时间：">
          <el-date-picker
            v-model="scanForm.smsj"
            type="daterange"
            value-format="yyyy-MM-dd"
            start-placeholder="开始日期"
            range-separator="至"
            end-placeholder="结束日期">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="进入影像时间：">
          <el-date-picker
            v-model="scanForm.jryxsj"
            type="daterange"
            value-format="yyyy-MM-dd"
            start-placeholder="开始日期"
            range-separator="至"
            end-placeholder="结束日期">
          </el-date-picker>
        </el-form-item>
        <el-form-item label="扫描人名称：">
          <el-input v-model="scanForm.smrmc"></el-input>
        </el-form-item>
      </el-form>
      <el-button class="searchButton" type="primary" @click="pageNum = 1,search()">查询</el-button>
    </div>
    <!-- 表格组件 -->
    <!-- eslint-disable -->
    <div class="tableWraper">
      <BasicTable 
              :tableData="tableData" 
              :tableOptions="tableOptions" 
              :treeType="treeType"
              :fatherToDetails="fatherToDetails">
        <template slot="customButton" slot-scope="customButtonData">
            <span
                type="primary" 
                :class="JSON.parse(customButtonData.currentTravel).ppzt =='未匹配'? 'customButton': 'pped' "
                @click="customButtonClick(customButtonData)">
                {{JSON.parse(customButtonData.customButtonData)[0].text}}
            </span>
        </template>
      </BasicTable>
    </div>
    <!-- eslint-enable -->
    <!-- 底部分页 -->
    <div class="foot_pagination">
      <div class="paginationWraper">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="pageNum"
          :page-sizes="[10, 20, 30, 40]"
          :page-size="100"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </div>
  </div>
  </div>
</template>
<script>
import BasicTable from '@/views/components/basic-table.vue';
import {
  queryList,
  queryDetails,
  matchList,
  uploadFile,
  phoneUpload,
  addImg,
} from '@/api/accountImg/index.js';
import { formatDate } from '@/assets/utils/timefn.js';

export default {
    components: {
      BasicTable,
    },
    data() {
      return {
        title: '扫描批次',
        scanForm: {
          dzbh: '',
          yxzt: '全部',
          ppzt: '',
          smsj: [],
          jryxsj: [],
          smrmc: '',
        },
        yxztArr: [{
            name: '全部',
            id: ''
          }, {
            name: '待扫描',
            id: '00'
          }, {
            name: '待确认',
            id: '01'
          }, {
            name: '已确认',
            id: '02'
          }, {
            name: '打回重扫',
            id: '03'
        }],
        ppztArr: [{
          name: '全部',
            id: ''
          }, {
            name: '未匹配',
            id: '0'
          }, {
            name: '自动匹配',
            id: '1'
          }, {
            name: '手动匹配',
            id: '2'
        }],
        tableData: [],
        tableOptions: [{
            label: '匹配状态',
            prop: 'ppzt',
          }, {
              label: '单证编号',
              prop: 'dzbh',
          }, {
              label: '进入影像时间',
              prop: 'jryxsj',
          }, {
              label: '扫描时间',
              prop: 'smsj',
          }, {
              label: '影像状态',
              prop: 'yxzt',
          }, {
              label: '扫描人',
              prop: 'smr',
          }, {
              label: '匹配人',
              prop: 'ppr',
          }, {
              label: '匹配时间',
              prop: 'ppsj',
          }],
        treeType: {
            type: 'operate',
            select: false,
            detail: true,
            customButton: [{
              text: '匹配',
            }],
        },
        pageNum: 1,
        pageSize: 10,
        total: 0,
      };
    },
    mounted() {
      this.init();
    },
    methods: {
      init() {
        this.search();
      },
      async search() {
        let ppztArr = {
          '0': '未匹配',
          '1': '自动匹配',
          '2': '手动匹配',
        };
        let yxztArr = {
          '00': '待扫描',
          '01': '待确认',
          '02': '已确认',
          '03': '打回重扫',
        }
        let params = {...this.scanForm};
        params.smsjStart = !!params.smsj ? params.smsj[0] : '';
        params.smsjEnd = !!params.smsj ? params.smsj[1] : '';
        params.jryxsjStart = !!params.jryxsj ? params.jryxsj[0] : '';
        params.jryxsjEnd = !!params.jryxsj ? params.jryxsj[1] : '';
        delete params.smsj;
        delete params.jryxsj;
        params.yxzt = params.yxzt === '全部' ? '' : params.yxzt;
        for(let item in yxztArr) {
          if (item === params.yxzt) {
            params.yxzt = item;
          }
        }
        params.pageNum = this.pageNum;
        params.pageSize = this.pageSize;
        let res = await queryList(params);
        if (res.data.code === '0000') {
          if (res.data.data.data.length) {
            this.tableData = res.data.data.data;
            this.total = res.data.data.dataTotalNum *1;
            this.tableData = this.tableData.map(item => {
              item.ppzt = ppztArr[item.ppzt];
              item.yxzt = yxztArr[item.yxzt];
              item.ppsj = formatDate(item.ppsj);
              item.jryxsj = formatDate(item.jryxsj);
              item.smsj = formatDate(item.smsj);
              return item;
            })
          } else {
            this.tableData = [];
            this.total = 0;
          }
        } else {
          this.$$message.error(res.data.message);
        }
      },
      async fatherToDetails(row, index) {
        if(row.ppzt === '未匹配') {
          this.$$message.warning('请先进行匹配！');
          return;
        }
        let params = {...row};
        params.origin = 'bzyx';
        this.$router.push({ path: '/imageCenter/scan/details', query: params });
      },
      async customButtonClick(data) {
        if (JSON.parse(data.currentTravel).ppzt != '未匹配') {
          this.$$message.warning('已匹配，请勿重复操作！');
          return;
        }
        let params = {...JSON.parse(data.currentTravel)}
        this.$router.push({ path: '/imageCenter/account/match', query: params});
      },
      handleSizeChange(val) {
        this.pageSize = val;
        this.search();
      },
      handleCurrentChange(val) {
        this.pageNum = val;
        this.search();
      },
    },
};
</script>
<style lang="scss">
.el-icon-circle-close{
  line-height: 22px!important;
}
</style>
<style lang="scss" scoped>
.el-input{
  width:164px;
}
.el-select{
  width:150px;
}
.el-date-editor{
  // width:198px;
  margin-right:20px;
}
/deep/.el-input__inner{
  height:28px;
}
.el-form-item{
  display:inline-block;
  margin-bottom: 0px!important;
}
.customButton{
  // margin-left:15px;
  color:#229FFF;
  cursor:pointer;
}
.pped{
  color: gray!important;
}
.scan{
  position:relative;
  height:100%;
  box-sizing:border-box;
  padding-top:20px;
  .searchBox{
    position:relative;
    // overflow:hidden;
    .searchButton{
      position: absolute;
      right:20px;
      top: 15px;
    }
  }
  .tableWraper{
    padding: 0 12px;
  }
  .foot_pagination{
    width:100%;
    border-top:1px solid #CCCCCC;
    .paginationWraper{
      padding: 10px 0;
      text-align:right;
    }
  }
}
</style>
