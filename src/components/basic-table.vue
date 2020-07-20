<template>
    <div class="basicTable">
        <el-table
            :data="tableData"
            border
            ref="commonTable"
            element-loading-text="正在拼命加载中"
            header-align="center"
            @selection-change="handleSelectionChange"
            :empty-text="blankWarn">
            <el-table-column
              type="selection"
              v-if="treeType.select"
              :selectable="selectCtrl"
              width="55"
              fixed>
            </el-table-column>
             <el-table-column
              v-if="treeType.showIndex&&!treeType.isPage"
              label="序号"
              type="index"
              width="50"
              align="center">
            </el-table-column>
            <el-table-column
              v-if="treeType.showIndex&&treeType.isPage"
              label="序号"
              type="index"
              width="50"
              align="center">
              <template slot-scope="scope">
                  <span>{{($parent.pageNum - 1) * $parent.pageSize + scope.$index + 1}}</span>
              </template>
            </el-table-column>
            <el-table-column
                v-for="(item, index) in tableOptions"
                :key="index"
                :width="item.width"
                :min-width="item.minWidth ? item.minWidth : '0'"
                :label="item.label"
                :prop="item.prop"
                align="center"
                show-overflow-tooltip>
                <template slot="header" v-if="item.filter">
                    <span>{{ item.label }}</span>
                    <i class="el-icon-arrow-down"></i>
                </template>
                <template slot-scope="scope" >
                  <!-- 表格展开 -->
                  <span v-if="index === 0 && treeType.expand">
                    <i
                      class="el-icon-arrow-up tableExpand"
                      v-show="scope.row.tableExpand"
                      @click="expandTab(scope.row,'1',scope.$index)"
                    ></i>
                    <!-- 表格收起 -->
                    <i
                      class="el-icon-arrow-down tableExpand"
                      v-show="!scope.row.tableExpand"
                      @click="expandTab(scope.row,'0',scope.$index)"
                    ></i>
                  </span>
                  <!-- eslint-disable -->
                  <!-- 表格内数据 -->
                  <!-- <span>{{scope.row[item.prop]}}</span> -->
                  <span>{{(item.prop =='customColumn' ? '' : scope.row[item.prop]) |emptyFilter}}</span>
                  <span>{{(item.prop.slice(0,4) =='sbwz' ? scope.row[item.prop.slice(0,4)] : '')}}</span>
                  <!-- <span>{{item}}</span> -->
                  <!-- 自定义列内容 -->
                  <span v-if="item.prop === 'customColumn'">
                    <template>
                      <slot name="customColumn" :customColumnData="JSON.stringify(scope.row)" :currentIndex="scope.$index"></slot>
                    </template>
                  </span>
                  <!-- eslint-enable -->
                </template>
                <div v-if="item.father">
                    <el-table-column
                        v-for="(j,jdex) in item.children"
                        :key="jdex"
                        :label="j.label"
                        align="center"
                        show-overflow-tooltip>
                        <template slot-scope="scope">
                          {{j.prop == 'amount' ? '￥' : '' +scope.row[j.prop]}}
                        </template>
                    </el-table-column>
                </div>
            </el-table-column>
            <el-table-column
                align="center"
                label="操作"
                :width="treeType.operateWidth&&treeType.operateWidth||'120px'"
                fixed="right"
                v-if="treeType.type === 'operate'">
                <template slot-scope="props">
                    <a
                        v-if="treeType.detail"
                        @click="toDetail(props.row, props.$index)"
                        :class="props.row.ppzt == '未匹配' ? 'disabledBtn' : 'czBtn'"
                        >详情</a>
                    <a
                        v-if="treeType.del"
                        @click="toDel(props.row, props.$index)"
                        class="czBtn"
                        >删除</a>
                        <!-- :class="props.row.dzCount > 0 || props.row.yxCount > 0?'grayEdit': ''" -->
                    <a
                        v-if="treeType.edit"
                        @click="toEdit(props.row, props.$index)"
                        class="czBtn"
                        >编辑</a>
                    
                        <!-- @click="toInstall(props.row, props.$index)" -->
                    <a
                        v-if="treeType.install"
                        @click="toInstall(props.row, props.$index)"
                        class="czBtn"
                        >下载</a>
                      <a
                        v-if="treeType.check"
                        @click="toCheck(props.row, props.$index)"
                        class="czBtn"
                        >查验</a>
                       <a
                        v-if="treeType.replace"
                        class="czBtn"
                        style="position: relative"
                        >
                          <input type="file" class="file" ref="replaceFile" title="" @change="toReplace($event,props.row, props.$index)">
                          <span style="cursor: pointer;">上传替换</span>
                        </a>
                    <a
                        v-if="treeType.forbid"
                        @click="toForbid(props.row, props.$index)"
                        class="czBtn"
                        >{{props.row.status === '0' ? '启用' : '禁用'}}</a>
                    <template v-show="treeType.customButton" >
                      <!-- eslint-disable -->
                      <slot name="customButton" :customButtonData="JSON.stringify(treeType.customButton)" :currentTravel="JSON.stringify(props.row)">
                      </slot>
                      <!-- eslint-enable -->
                    </template>
                </template>
            </el-table-column>
            <el-table-column v-if="treeType.expand" type="expand" width="1">
              <template slot-scope="props">
                <slot name="expand" :expandData="JSON.stringify(props.row)"></slot>
              </template>
            </el-table-column>
        </el-table>
    </div>
</template>
<script>
export default {
    props: {
        treeType: Object,
        tableData: Array,
        tableOptions: Array,
        fatherDel: Function,
        fatherEdit: Function,
        fatherInstall: Function,
        fatherForbid: Function,
        fatherToDetails: Function,
        fatherSelectChange: Function,
        fatherCheck:Function,
        fatherReplace:Function
    },
    data() {
        return {
            blankWarn: '暂无数据',
            forbidText: '禁用',
            selectedRow: [],
        };
    },
    methods: {
      /*eslint-disable*/
      expandTab(row, control, index) {
        row.tableExpand = !row.tableExpand;
        this.toogleExpand(row, control, index);
      },
      /* eslint-enable */
          // 展开行
          /*eslint-disable*/
      toogleExpand(row, control, rowIndex) {
        let $table = this.$refs.commonTable;
        // this.enterListData.map(item => {
          //   if (row.id != item.id && rowIndex != 0) {
            //     $table.toggleRowExpansion(item, false);
        //   }
        // });
        if (control === '1') {
          $table.toggleRowExpansion(row, false);
        } else {
          // this.getInsideList(rowIndex, row.ssflmc)
          $table.toggleRowExpansion(row, true);
        }
      },
      handleSelectionChange(val) {
        console.log('点击表格选择', val);
        if (!this.fatherSelectChange) return;
        this.selectedRow = val;
        this.fatherSelectChange(val);
      },
      selectCtrl(row,index) {
        return true
        // console.log('selectCtrlllllll', row,index,this.selectedRow)
        // return this.selectedRow.length ? row.id === this.selectedRow[0].id : true;
      },
    /* eslint-enable */
      toDetail(row, rowIndex) {
        console.log('去详情！');
        if (!this.fatherToDetails) return;
        this.fatherToDetails(row, rowIndex);
      },
      // 删除
      toDel(row, rowIndex) {
          if (!this.fatherDel) return;
          this.fatherDel(row, rowIndex);
      },
      // 编辑
      toEdit(row, rowIndex) {
          if (!this.fatherEdit) return;
          this.fatherEdit(row, rowIndex);
      },
      //下载
      toInstall(row, rowIndex){
        if (!this.fatherInstall) return;
        this.fatherInstall(row, rowIndex);
      },
      //查验
      toCheck(row, rowIndex){
        if (!this.fatherCheck) return;
        this.fatherCheck(row, rowIndex);
      },
      toReplace(e,row, rowIndex){
         if (!this.fatherReplace) return;
        this.fatherReplace(e,row, rowIndex);
      },
      // 禁用
      toForbid(row, rowIndex) {
          if (!this.fatherForbid) return;
          this.fatherForbid(row, rowIndex);
      },
    },
    filters: {
        emptyFilter(val) {
          // console.log(val)
          if (!val) return '/';
          return val;
        },
        customButtonClick(row, rowIndex, d) {
            if (!this.treeType.customButton) return;
            this.treeType.customButton[d].customButtonClick(row, rowIndex);
        },
    }
};
</script>
<style lang="scss">

.czBtn{
  color:#229FFF;
  cursor:pointer;
  margin-right:15px;
}
.grayEdit{
  color: #666;
}
</style>
<style lang="scss" scoped>
.disabledBtn{
  color: gray!important;
  margin-right:15px;
}
  .basicTable{
    .tableExpand{
      // float: left;
      // margin-left: 5px;
      margin-right: 15px;
      cursor: pointer;
    }
    .el-form-item{
      margin-bottom: 0!important;
      display: block!important;
    }
  }
  .file {
  cursor: pointer;
  position: absolute;
  left: 0;
  top: 0;
  bottom: 0;
  right: 0;
  width: 100%;
  opacity: 0;
  font-size: 0;
}
</style>
