<template>
  <div>
    <h2><i class="el-icon-folder-opened"></i>&nbsp;多图片全检测</h2>
    <el-form ref="form" :model="form" label-width="100px">
      <el-form-item label="文件夹路径">
        <el-input v-model="form.name" class="path-area" :disabled="true"></el-input>
        <el-button type="primary" plain @click="getFolderPath">选择文件夹</el-button>
        <el-tooltip :content="'图像显示时按任意键跳过当前图像'" placement="top">
          <el-switch
              style="margin-left: 10px"
              v-model="show_img"
              active-text="展示图像"
              inactive-text="不展示">
          </el-switch>
        </el-tooltip>
      </el-form-item>
      <div class="block" style="margin-left: 20px; margin-right: 20px">
        <span class="demonstration">调整置信度阈值</span>
        <el-slider
            style="margin-top: 10px"
            v-model="slider_value"
            :format-tooltip="formatTooltip"
            @input="slider_change($event)">
        </el-slider>
      </div>
      <div v-if="vis">
        <p>检测成功！本次送检共包含样本{{ res.sample_sum }}个</p>
        <li>香农·威纳指数为:&nbsp;<b>{{ res.sw_index }}</b>&nbsp;[现实中通常为1.5-3.5之间]</li>
        <li>物种丰富度指数为:&nbsp;<b>{{ res.sr_index }}</b>&nbsp;[0-1之间]</li>
        <p>藻类样本按照送检占比从高到低排序</p>
        <el-table
            :data="res.sorted_samples"
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
      </div>
      <div class="button-area">
        <el-form-item>
          <el-button type="primary" @click="onSubmit">立即检测</el-button>
          <el-button @click="resetForm">重置路径</el-button>
        </el-form-item>
      </div>
      <div v-if="start_det" class="progress_circle">
        <el-progress type="circle" :percentage="progress" style="margin-right: 100px"></el-progress>
      </div>
    </el-form>
  </div>
</template>

<script>
const baseURL = 'http://localhost:8090'
export default {
  name: "FolderDet",
  data() {
    return {
      progress: 0,
      slider_value: 25,
      form: {name: ''},
      vis: false,
      start_det: false,
      show_img: false,
      res: {
        // det_time:'',
        sample_sum: 0,
        sw_index: 0,
        sr_index: 0,
        sorted_samples: [],
        // records: [],
        // detail_record: {},
      },
    }
  },
  methods: {
    onSubmit() {
      let self = this;
      let path = this.form.name;
      this.start_det = true;
      this.$http.post(baseURL + '/detect_folder',
          {
            path: path,
            conf_th: this.slider_value / 100,
            show_img: this.show_img
          })
          .then(response => {
            // console.log(response.data)
            let data = response.data;
            self.res.det_time = data.det_time;
            self.res.sample_sum = data.sample_sum;
            self.res.sorted_samples = data.sorted_samples;
            self.res.sw_index = data.SW_index;
            self.res.sr_index = data.SR_index;
            self.vis = true;
            self.progress = 100;
            self.start_det = false;
            self.progress = 0;
          }).catch(error => {
        console.log(error)
      });
      this.vis = false;
      this.$http.post(baseURL + '/countImages',
          {
            path: path,
          })
          .then(response => {
            // console.log(response.data)
            let data = response.data;
            let count = data['count'];  // 图像数
            for (let i = 0; i < count; i++) {
              self.sleep(50).then(()=>{
                self.progress += Math.floor(1.0/count*100.0);
                // console.log(self.progress)
              });
              if(self.progress>=80){
                break;
              }
            }
          }).catch(error => {
        console.log(error)
      });
    },
    getFolderPath() {
      let path = this.chooseFolderPath();
      this.form.name = path
    },
    resetForm() {
      this.form.name = ''
    },
    chooseFolderPath() {
      const {remote} = require('electron')
      const result = remote.dialog.showOpenDialog({
        properties: ['openDirectory'],
      });
      if (result !== undefined) {
        return result[0]
      } else {
        return ''
      }
    },
    slider_change(val) {
      this.slider_value = val
    },
    formatTooltip(val) {
      return val / 100;
    },
    sleep(ms) {
      return new Promise(resolve =>
          setTimeout(resolve, ms)
      )
    }
  }
}
</script>

<style scoped>
.button-area{
  margin-top: 20px;
  margin-left: 150px;
}

.progress_circle{
  margin-top: 20px;
  margin-left: 275px;
}
</style>