<template>
  <div>
    <h2><i class="el-icon-aim"></i>&nbsp;指定藻种检测</h2>
    <el-form ref="form" :model="form" label-width="85px">
      <el-form-item label="样本路径">
        <el-cascader
            placeholder="试试搜索：硅藻门"
            v-model="category"
            :options="options"
            filterable>
        </el-cascader>
        <el-input v-model="form.name" class="path-area" :disabled="true"></el-input>
        <el-button type="primary" plain @click="getFolderPath">选择样本</el-button>
      </el-form-item>
      <div class="block" style="margin-bottom: 20px; margin-left: 10px">
        <el-tooltip :content="'图像显示时按任意键跳过当前图像'" placement="top">
          <el-switch
              style="margin-left: 10px"
              v-model="show_img"
              active-text="展示图像"
              inactive-text="不展示">
          </el-switch>
        </el-tooltip>
      </div>
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
        <p>检测成功！本次送检共包含指定样本{{res.sample_sum}}个</p>
        <el-table
            :data="res.sorted_samples"
            stripe
            style="width: 100%">
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
              label="数量">
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
  name: "TargetDet",
  data(){
    return{
      form:{name: ''},
      vis: false,
      show_img: false,
      slider_value: 25,
      progress: 0,
      start_det: false,
      res:{det_time:'',
        sample_sum:0,
        sorted_samples:[]
      },
      category:[],
      options: [
        {
          value: 'GUI',
          label: '硅藻门',
          children: [
            {value: '01008000', label: '舟形藻属'},
            {value: '01010000', label: '异极藻属（异端藻属）'},
            {value: '01011000', label: '桥弯藻属'},
            {value: '01009000', label: '羽纹藻属'},
            {value: '01004000', label: '脆杆藻属'},
            {value: '01006000', label: '卵形藻属'},
            {value: '01026000', label: '海链藻属'},
            {value: '01034001', label: '双尾藻属'},
            {value: '01013000', label: '双菱藻属'},
            {value: '01018000', label: '布纹藻属（双缝藻属）'},
            {value: '01007000', label: '辐节藻属'},
            {value: '01023001', label: '波缘藻属(波纹藻属)'},
            {value: '01039001', label: '海线藻属'},
            {value: '01005000', label: '针杆藻属'},
            {value: '01033000', label: '盒形藻属'},
            {value: '01024000', label: '圆筛藻属'},
            {value: '01020000', label: '平板藻属'},
            {value: '01015009', label: '菱形藻属'},
            {value: '01037001', label: '弯角藻属'},
            {value: '01049001', label: '齿状藻属'},
            {value: '01002000', label: '小环藻属'},
            {value: '01069000', label: '斜纹藻属'},
          ]
        },
        {
          value: 'LAN',
          label: '蓝藻门',
          children: [
            {value: '02012000', label: '颤藻属'},
            {value: '02009000', label: '鱼腥藻属'},
            {value: '02008000', label: '念珠藻属'},
            {value: '02002000', label: '微囊藻属'},
            {value: '02013000', label: '席藻属'},
            {value: '02001000', label: '色球藻属'},
            {value: '02022000', label: '鞘丝藻属'},
          ]
        },
        {
          value: 'LV',
          label: '绿藻门',
          children: [
            {value: '10076000', label: '凹顶鼓藻属'},
            {value: '10082004', label: '棒形鼓藻属'},
            {value: '10217016', label: '单针藻属'},
            {value: '10165001', label: '独球藻属'},
            {value: '10088000', label: '多棘鼓藻属'},
            {value: '10024002', label: '多芒藻属'},
            {value: '10045000', label: '鼓藻属'},
            {value: '10116000', label: '红球藻属'},
            {value: '10034000', label: '胶网藻属（网球藻属）'},
            {value: '10043000', label: '角星鼓藻属'},
            {value: '10176000', label: '克里藻属'},
            {value: '10009000', label: '空球藻属'},
            {value: '10039000', label: '空星藻属'},
            {value: '10114001', label: '螺带鼓藻属'},
            {value: '10003001', label: '绿梭藻属'},
            {value: '10079000', label: '毛鞘藻属'},
            {value: '10037000', label: '盘星藻属'},
            {value: '10004000', label: '盘藻属'},
            {value: '10080000', label: '鞘藻属'},
            {value: '10057000', label: '肾形藻属'},
            {value: '10008001', label: '实球藻属'},
            {value: '10122000', label: '双星藻属'},
            {value: '10064001', label: '双形藻属'},
            {value: '10127000', label: '水绵属'},
            {value: '10042000', label: '丝藻属'},
            {value: '10055000', label: '梭形鼓藻属'},
            {value: '10181000', label: '微孢藻属'},
            {value: '10075000', label: '微星鼓藻属(小星藻属)'},
            {value: '10044000', label: '新月藻属'},
            {value: '10001000', label: '衣藻属'},
            {value: '10095000', label: '圆丝鼓藻属'},
            {value: '10033000', label: '月牙藻属'},
            {value: '10038000', label: '栅藻属'},
            {value: '10053003', label: '竹枝藻属'},
            {value: '10083006', label: '柱形鼓藻属'},
            {value: '10054000', label: '转板藻属'},
          ]
        },
      ],
    }
  },
  methods: {
    onSubmit(){
      let self = this;
      let path = this.form.name;
      this.start_det = true;
      let target = this.category[1];
      this.$http.post(baseURL + '/det_target', {path: path, target: target, conf_th: this.slider_value / 100, show_img: this.show_img})
          .then(response=>{
            console.log(response.data)
            let data = response.data;
            self.res.det_time = data.det_time;
            self.res.sample_sum = data.sample_sum;
            self.res.sorted_samples = data.sorted_samples;
            self.vis = true;
            self.progress = 100;
            self.start_det = false;
            self.progress = 0;
          }).catch(error=>{
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
    resetForm(){
      this.form.name=''
    },
    getFolderPath(){
      let path = this.chooseFolderPath();
      this.form.name = path
    },
    /*tool*/
    chooseFolderPath(){
      const {remote} = require('electron')
      const result = remote.dialog.showOpenDialog({
        properties:['openDirectory'],
      });
      if (result !== undefined){
        return result[0]
      }
      else {
        return ''
      }
    },
    slider_change(val) {
      this.slider_value = val
    },
    formatTooltip(val) {
      return val / 100;
    },
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