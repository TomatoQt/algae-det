<template>
  <div id="wrapper">
    <el-tabs :tab-position=tabPosition style="height: auto;" @tab-click="handleTabsChange">
      <el-tab-pane label="多图片全检测">
        <folder-det></folder-det>
      </el-tab-pane>

      <el-tab-pane label="单图片全检测">
        <image-det></image-det>
      </el-tab-pane>

      <el-tab-pane label="指定藻种检测">
        <target-det></target-det>
      </el-tab-pane>

      <el-tab-pane label="检测记录查看" name="records" v-loading.fullscreen.lock="fullscreenLoading">
        <h2>
          <i class="el-icon-discover"></i>
          检测记录查看
        </h2>
        <el-table
            :data="res.records"
            style="width: 100%">
          <el-table-column
              type="index"
              :index="indexMethod">
          </el-table-column>
          <el-table-column
              prop="now_time"
              sortable
              label="操作时间"
              width="180">
          </el-table-column>
          <el-table-column
              prop="func"
              label="类型"
              width="180"
              :filters="[{text: '多图片全检测', value: '多图片全检测'}, {text: '单图片全检测', value: '单图片全检测'}, {text: '指定藻种检测', value: '指定藻种检测'}]"
              :filter-method="filterHandler">
          </el-table-column>
          <el-table-column
              prop="save_dir"
              label="记录位置">
          </el-table-column>
          <el-table-column
              fixed="right"
              label="操作"
              width="150">
            <template slot-scope="scope">
              <el-button type="success" plain @click="open_folder(scope.row)" size="small">打开文件夹</el-button>
              <el-button @click="checkRecordDetail(scope.row);dialogVisible=true" type="text" size="small">查看记录详情
              </el-button>
              <el-dialog
                  title="记录详情"
                  :visible.sync="dialogVisible"
                  width="80%"
                  append-to-body>
                <p>上次送检共包含样本{{ res.detail_record.sample_sum }}个</p>
                <p>上次送检时间为{{ res.detail_record.now_time }}</p>
                <li>香农·威纳指数为:&nbsp;<b>{{res.detail_record.Shannon_Wiener_Index}}</b>&nbsp;[现实中通常为1.5-3.5之间]</li>
                <li>物种丰富度指数为:&nbsp;<b>{{res.detail_record.species_richness_index}}</b>&nbsp;[0-1之间]</li>
                <p>藻类样本按照送检占比从高到低排序</p>
                <el-table
                    :data="res.detail_record.sorted_CN_samples"
                    stripe
                    style="width: 100%">
                  <el-table-column
                      prop="rank"
                      label="排序"
                      width="50">
                  </el-table-column>
                  <el-table-column
                      prop="door"
                      label="门"
                      width="180">
                  </el-table-column>
                  <el-table-column
                      prop="mouse"
                      label="属">
                  </el-table-column>
                  <el-table-column
                      prop="number"
                      label="数量" width="50">
                  </el-table-column>
                  <el-table-column
                      prop="percent"
                      label="占比(%)">
                  </el-table-column>
                </el-table>
                <span slot="footer" class="dialog-footer">
                  <el-button type="primary" @click="dialogVisible = false">退 出</el-button>
                  <el-button @click="export2Excel(record_path)">导出为表格</el-button>
                  <el-button @click="openExcel(record_path)">打开表格</el-button>
                </span>
              </el-dialog>
            </template>
          </el-table-column>
        </el-table>
        <div style="margin-top: 20px; float: right; margin-right: 40px">
          <el-button @click="exportAll">全部导出为表格</el-button>
        </div>
      </el-tab-pane>
    </el-tabs>
  </div>
</template>

<script>
const baseURL = 'http://localhost:8090'
import ImageDet from "./LandingPage/ImageDet";
import TargetDet from "./LandingPage/TargetDet";
import FolderDet from "./LandingPage/FolderDet";

export default {
  name: 'landing-page',
  components: {ImageDet, TargetDet, FolderDet},
  data() {
    return {
      dialogVisible: false,
      tabPosition: 'left',
      form: {name: ''},
      vis: false,
      fullscreenLoading: false,
      res: {
        det_time: '',
        sample_sum: 0,
        sorted_samples: [],
        records: [],
        detail_record: {},
      },
      record_path: '',
    }
  },
  methods: {
    handleTabsChange(tab, e) {
      if (tab.name === 'records') {
        this.get_record();
      }
    },
    get_record() {
      let self = this;
      this.fullscreenLoading = true;
      setTimeout(() => {
        this.fullscreenLoading = false;
      }, 500);
      this.$http.get(baseURL + '/load_all_record')
          .then(response => {
            let data = response.data;
            if (data['code'] === 0) {
              self.res.records = data['records'];
            } else {
              self.$message.error(data['msg']);
            }
          }).catch(error => {
        console.log(error)
      })
    },
    checkRecordDetail(row) {
      // console.log('test', row)
      let save_dir = row['save_dir'];
      let self = this;
      this.record_path = save_dir
      this.$http.post(baseURL + '/load_one_record', {save_dir: save_dir})
          .then(response => {
            // console.log(response.data)
            let data = response.data;
            self.res.detail_record = data['detail_record'];
          }).catch(error => {
        console.log(error)
      })
    },
    open_folder(row) {
      let save_dir = row['save_dir'];
      const shell = require('electron').shell;
      shell.showItemInFolder(save_dir + '\\label');
    },
    indexMethod(index) {
      return index + 1;
    },
    filterHandler(value, row, column) {
      const property = column['property'];
      return row[property] === value;
    },
    export2Excel(record_path) {
      // console.log('test', row)
      // let save_dir = row['save_dir'];
      let self = this;
      this.$http.post(baseURL + '/export2Excel', {save_dir: record_path})
          .then(response => {
            // console.log(response.data)
            let data = response.data
            if (data['code'] === 0) {
              self.$message.success('表格导出成功!');
            } else {
              self.$message.error('表格导出失败,请联系管理员!');
            }
          }).catch(error => {
        console.log(error)
      })
    },
    exportAll() {
      let self = this;
      this.$http.get(baseURL + '/exportAll')
          .then(response => {
            let data = response.data
            if (data['code'] === 0) {
              self.$message.success('表格全部导出成功!');
            } else {
              self.$message.error('表格导出故障,请联系管理员!');
            }
          }).catch(error => {
        console.log(error)
      })
    },
    openExcel(record_path) {
      let self = this;
      const shell = require('electron').shell;
      this.$http.post(baseURL + '/checkXlsx', {save_dir: record_path})
          .then(response => {
            let data = response.data
            if (data['code'] === 0) {
              let xlsx_path = data['xlsx_path']
              // console.log(xlsx_path)
              shell.openItem(xlsx_path)
            } else {
              self.$message.error('表格尚未导出,请导出后重试!');
            }
          }).catch(error => {
        console.log(error)
      })
    }
  }
}
</script>

<style>
.path-area {
  width: 300px;
}
</style>
