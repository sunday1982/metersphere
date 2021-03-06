<template>
  <div v-loading="result.loading" class="pressure-config-container">
    <el-row>
      <el-col :span="10">
        <el-form :inline="true">
          <el-form-item>
            <div class="config-form-label">{{ $t('load_test.thread_num') }}</div>
          </el-form-item>
          <el-form-item>
            <el-input-number
              :disabled="true"
              :placeholder="$t('load_test.input_thread_num')"
              v-model="threadNumber"
              @change="calculateChart"
              :min="1"
              size="mini"/>
          </el-form-item>
        </el-form>
        <el-form :inline="true">
          <el-form-item>
            <div class="config-form-label">{{ $t('load_test.duration') }}</div>
          </el-form-item>
          <el-form-item>
            <el-input-number
              :disabled="true"
              :placeholder="$t('load_test.duration')"
              v-model="duration"
              :min="1"
              @change="calculateChart"
              size="mini"/>
          </el-form-item>
        </el-form>
        <el-form :inline="true">
          <el-form-item>
            <el-form-item>
              <div class="config-form-label">{{ $t('load_test.rps_limit') }}</div>
            </el-form-item>
            <el-form-item>
              <el-switch v-model="rpsLimitEnable"/>
            </el-form-item>
            <el-form-item>
              <el-input-number
                :disabled="true"
                :placeholder="$t('load_test.input_rps_limit')"
                v-model="rpsLimit"
                @change="calculateChart"
                :min="1"
                size="mini"/>
            </el-form-item>
          </el-form-item>
        </el-form>

        <el-form :inline="true" class="input-bottom-border">
          <el-form-item>
            <div>{{ $t('load_test.ramp_up_time_within') }}</div>
          </el-form-item>
          <el-form-item>
            <el-input-number
              :disabled="true"
              placeholder=""
              :min="1"
              :max="duration"
              v-model="rampUpTime"
              @change="calculateChart"
              size="mini"/>
          </el-form-item>
          <el-form-item>
            <div>{{ $t('load_test.ramp_up_time_minutes') }}</div>
          </el-form-item>
          <el-form-item>
            <el-input-number
              :disabled="true"
              placeholder=""
              :min="1"
              :max="Math.min(threadNumber, rampUpTime)"
              v-model="step"
              @change="calculateChart"
              size="mini"/>
          </el-form-item>
          <el-form-item>
            <div>{{ $t('load_test.ramp_up_time_times') }}</div>
          </el-form-item>
        </el-form>
      </el-col>
      <el-col :span="14">
        <div class="title">{{ $t('load_test.pressure_prediction_chart') }}</div>
        <ms-chart class="chart-container" ref="chart1" :options="orgOptions" :autoresize="true"></ms-chart>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import echarts from "echarts";
import MsChart from "@/business/components/common/chart/MsChart";

const TARGET_LEVEL = "TargetLevel";
const RAMP_UP = "RampUp";
const STEPS = "Steps";
const DURATION = "duration";
const RPS_LIMIT = "rpsLimit";
const RPS_LIMIT_ENABLE = "rpsLimitEnable";

export default {
  name: "MsPerformancePressureConfig",
  components: {MsChart},
  props: ['report'],
  data() {
    return {
      result: {},
      threadNumber: 10,
      duration: 10,
      rampUpTime: 10,
      step: 10,
      rpsLimit: 10,
      rpsLimitEnable: false,
      orgOptions: {},
    }
  },
  mounted() {
    this.getLoadConfig();
  },
  methods: {
    calculateLoadConfiguration: function (data) {
      data.forEach(d => {
        switch (d.key) {
          case TARGET_LEVEL:
            this.threadNumber = d.value;
            break;
          case RAMP_UP:
            this.rampUpTime = d.value;
            break;
          case DURATION:
            this.duration = d.value;
            break;
          case STEPS:
            this.step = d.value;
            break;
          case RPS_LIMIT:
            this.rpsLimit = d.value;
            break;
          default:
            break;
        }
      });

      this.threadNumber = this.threadNumber || 10;
      this.duration = this.duration || 30;
      this.rampUpTime = this.rampUpTime || 12;
      this.step = this.step || 3;
      this.rpsLimit = this.rpsLimit || 10;

      this.calculateChart();
    },
    getLoadConfig() {
      if (!this.report.id) {
        return;
      }
      this.$get("/performance/report/" + this.report.id, res => {
        let data = res.data;
        if (data) {
          if (data.loadConfiguration) {
            let d = JSON.parse(data.loadConfiguration);
            this.calculateLoadConfiguration(d);
          } else {
            this.$get('/performance/get-load-config/' + this.report.testId, (response) => {
              if (response.data) {
                let data = JSON.parse(response.data);
                this.calculateLoadConfiguration(data);
              }
            });
          }
        } else {
          this.$error(this.$t('report.not_exist'))
        }
      });
    },
    calculateChart() {
      if (this.duration < this.rampUpTime) {
        this.rampUpTime = this.duration;
      }
      if (this.rampUpTime < this.step) {
        this.step = this.rampUpTime;
      }
      this.orgOptions = {
        xAxis: {
          type: 'category',
          boundaryGap: false,
          data: []
        },
        yAxis: {
          type: 'value'
        },
        tooltip: {
          trigger: 'axis',
          formatter: '{a}: {c0}',
          axisPointer: {
            lineStyle: {
              color: '#57617B'
            }
          }
        },
        series: [{
          name: 'User',
          data: [],
          type: 'line',
          step: 'start',
          smooth: false,
          symbolSize: 5,
          showSymbol: false,
          lineStyle: {
            normal: {
              width: 1
            }
          },
          areaStyle: {
            normal: {
              color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                offset: 0,
                color: 'rgba(137, 189, 27, 0.3)'
              }, {
                offset: 0.8,
                color: 'rgba(137, 189, 27, 0)'
              }], false),
              shadowColor: 'rgba(0, 0, 0, 0.1)',
              shadowBlur: 10
            }
          },
          itemStyle: {
            normal: {
              color: 'rgb(137,189,27)',
              borderColor: 'rgba(137,189,2,0.27)',
              borderWidth: 12
            }
          },
        }]
      };
      let timePeriod = Math.floor(this.rampUpTime / this.step);
      let timeInc = timePeriod;

      let threadPeriod = Math.floor(this.threadNumber / this.step);
      let threadInc1 = Math.floor(this.threadNumber / this.step);
      let threadInc2 = Math.ceil(this.threadNumber / this.step);
      let inc2count = this.threadNumber - this.step * threadInc1;
      for (let i = 0; i <= this.duration; i++) {
        // x 轴
        this.orgOptions.xAxis.data.push(i);
        if (i > timePeriod) {
          timePeriod += timeInc;
          if (inc2count > 0) {
            threadPeriod = threadPeriod + threadInc2;
            inc2count--;
          } else {
            threadPeriod = threadPeriod + threadInc1;
          }
          if (threadPeriod > this.threadNumber) {
            threadPeriod = this.threadNumber;
          }
          this.orgOptions.series[0].data.push(threadPeriod);
        } else {
          this.orgOptions.series[0].data.push(threadPeriod);
        }
      }
    },
  },
  watch: {
    report: {
      handler(val) {
        if (!val.testId) {
          return;
        }
        this.getLoadConfig();
      },
      deep: true
    }
  }
}
</script>


<style scoped>
.pressure-config-container .el-input {
  width: 130px;

}

.pressure-config-container .config-form-label {
  width: 130px;
}

.pressure-config-container .input-bottom-border input {
  border: 0;
  border-bottom: 1px solid #DCDFE6;
}

.chart-container {
  width: 100%;
}

.el-col .el-form {
  margin-top: 15px;
  text-align: left;
}

.el-col {
  margin-top: 15px;
  text-align: left;
}

.title {
  margin-left: 60px;
}
</style>
