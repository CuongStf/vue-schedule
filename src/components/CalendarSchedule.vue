<template>
  <div class="schedule">
    <el-row>
      <el-col
        v-if="isByDay"
        :span="24"
        class="flex-col flex__center"
      >
        <div class="flex-row flex__center">
          <el-button
            @click="prevDay"
            type="primary"
            size="mini"
            icon="el-icon-arrow-left"
          >
            Trước
          </el-button>
          <!-- <el-button
            plain
            disabled
            size="mini"
          >
            {{dateCurrentFormatUTC}}
          </el-button> -->
          <el-date-picker
            v-model="dateCurrent"
            type="date"
            size="mini"
            format="dd/MM/yyyy"
            placeholder="Chọn ngày hiển thị"
            prefix-icon="none"
            clearable
            @change="changeDate"
          >
          </el-date-picker>
          <el-button
            @click="nextDay"
            size="mini"
            type="primary"
          >
            Sau
            <i class="el-icon-arrow-right el-icon-right"></i>
          </el-button>
        </div>
        <el-table
          :data="listRoom"
          style="width: 100%"
          size="large"
          v-loading="loadingTable"
          :height="heightTable"
        >
          <el-table-column
            label="Nơi chốn"
            prop="place"
            width="150"
            fixed
            show-overflow-tooltip
          ></el-table-column>
          <el-table-column
            header-align="left"
            align="left"
            :width="listTimeline.length * 120"
          >
            <template slot="header">
              <div class="listTime">
                <div
                  v-for="time in listTimeline"
                  :key="time"
                  class="itemTime"
                >
                  {{time}}
                </div>
              </div>
            </template>
            <template slot-scope="scope">
              <div class="list-event">
                <div class="width__100 height__100">
                  <div
                    v-for="(event, index) in listEvents"
                    :key="index"
                    :class="['event-item', scope.row.id === event.place_id? '' : 'none' ]"
                    :style=" 'left:' + getDistanceToLeft(event.hoursStart) + 'px;' +
                  'width: ' + getWidthAEvent(event.hoursStart, event.hoursEnd) + 'px'"
                    @click="getEvent(event)"
                  >
                    <el-tooltip
                      effect="dark"
                      :content="event.name"
                      placement="top"
                    >
                      <div
                        v-if="scope.row.id === event.place_id"
                        :class="['width__100 height__100', customClassEventByDay]"
                      >
                        <h3 class="nameEvent">{{event.name}}</h3>
                        <div>{{getTimeFromDate(event.hoursStart)}} - {{getTimeFromDate(event.hoursEnd)}}</div>
                      </div>
                    </el-tooltip>
                  </div>
                </div>
              </div>
            </template>
          </el-table-column>
        </el-table>
      </el-col>
      <el-col
        v-else
        :span="24"
        class="flex-col flex__center"
      >
        <el-button-group>
          <el-button
            @click="prevWeek"
            type="primary"
            size="mini"
            icon="el-icon-arrow-left"
          >Tuần trước</el-button>
          <el-button
            @click="nextWeek"
            size="mini"
            type="primary"
          >Tuần tới<i class="el-icon-arrow-right el-icon-right"></i></el-button>
        </el-button-group>
        <el-table
          :data="listRoom"
          style="width: 100%"
          :height="heightTable"
          size="mini"
          v-loading="loadingTable"
        >
          <el-table-column
            label="Nơi chốn"
            prop="place"
            show-overflow-tooltip
            width="120"
            fixed="left"
          ></el-table-column>
          <el-table-column
            v-for="item in dayOfWeek"
            :key="item"
            :label="item"
            header-align="center"
            align="center"
          >
            <el-table-column
              label="Sáng"
              header-align="center"
              align="center"
              :width="widthSessionOfTheDay"
            >
              <template slot-scope="scope">
                <div
                  v-for="(event, index) in listEvents"
                  :key="index"
                  class="listEvent"
                >
                  <div
                    v-if="(event.place_id === scope.row.id) && (getDateFormatFromDate(event.date) === item) && (getHoursFromDate(event.hoursStart) < 12)"
                    :class="['event', customClassEventByWeek]"
                    @click="getEvent(event)"
                  >
                    <h3 class="name margin-0 padding-0">{{event.name}}</h3>
                    <div>{{getTimeFromDate(event.hoursStart)}} - {{getTimeFromDate(event.hoursEnd)}}</div>
                  </div>
                </div>
              </template>
            </el-table-column>
            <el-table-column
              label="Chiều"
              header-align="center"
              align="center"
              :width="widthSessionOfTheDay"
            >
              <template slot-scope="scope">
                <div
                  v-for="(event, index) in listEvents"
                  :key="index"
                  class="listEvent"
                >
                  <div
                    v-if="(event.place_id === scope.row.id) && (getDateFormatFromDate(event.date) === item) && (getHoursFromDate(event.hoursStart) > 12 || getHoursFromDate(event.hoursStart) == 12)"
                    :class="['event', customClassEventByWeek]"
                    @click="getEvent(event)"
                  >
                    <h3 class="name margin-0 padding-0">{{event.name}}</h3>
                    <div>{{getTimeFromDate(event.hoursStart)}} - {{getTimeFromDate(event.hoursEnd)}}</div>
                  </div>
                </div>
              </template>
            </el-table-column>
          </el-table-column>
        </el-table>
      </el-col>
    </el-row>
  </div>
</template>
<script>
import moment, { months } from 'moment'
export default {
  name: 'Schedule',
  props: {
    isByDay: {
      default: false,
      type: Boolean,
    },
    listTimeline: {
      default: function () {
        return ['08:00', '09:00', '10:00', '11:00', '12:00', '13:00', '14:00', '15:00', '16:00', '17:00'];
      },
      type: Array
    },
    listRoom: {
      default: function () {
        return [
          {
            id: 1,
            place: 'Phòng họp 12a.1'
          }, {
            id: 2,
            place: 'Phòng họp 12a.2'
          }, {
            id: 3,
            place: 'Phòng họp 12a.3'
          }, {
            id: 4,
            place: 'Phòng họp 12a.4'
          }, {
            id: 5,
            place: 'Phòng họp 12a.5'
          }, {
            id: 6,
            place: 'Phòng họp 12a.6'
          }, {
            id: 7,
            place: 'Phòng họp 12a.7'
          }, {
            id: 8,
            place: 'Phòng họp 12a.8'
          }, {
            id: 9,
            place: 'Phòng họp 12a.9'
          }, {
            id: 10,
            place: 'Phòng họp 12a.10'
          }, {
            id: 11,
            place: 'Phòng họp 12a.11'
          }
        ];
      },
      type: Array
    },
    listEvent: {
      default: function () {
        return [
          {
            place_id: 1,
            date: 1553045407,
            hoursStart: 1553045407,
            hoursEnd: 1553052607,
            name: 'Họp hội đồng quản trị thường niên Tổng công ty Mobifone'
          },
          {
            place_id: 3,
            date: 1553052607,
            hoursStart: 1553052607,
            hoursEnd: 1553059807,
            name: 'Họp hội đồng quản trị eoffice'
          },
          {
            place_id: 3,
            date: 1553045407,
            hoursStart: 1553045407,
            hoursEnd: 1553052607,
            name: 'Tình hình báo cáo nhân sự'
          },
          {
            place_id: 3,
            date: 1553665055,
            hoursStart: 1553665055,
            hoursEnd: 1553675855,
            name: 'Tình hình báo cáo nhân sự EOFFICE'
          },
          {
            place_id: 8,
            date: 1555816218,
            hoursStart: 1555816218,
            hoursEnd: 1555837818,
            name: 'Báo cáo tình hình phát triển trung tâm công nghệ thông tin Mobifone'
          },
          {
            place_id: 1,
            date: 1555816218,
            hoursStart: 1553045407,
            hoursEnd: 1553052607,
            name: 'Họp hội đồng quản trị thường niên Tổng công ty Mobifone'
          },
          {
            place_id: 3,
            date: 1555816218,
            hoursStart: 1553052607,
            hoursEnd: 1553059807,
            name: 'Họp hội đồng quản trị eoffice'
          },
          {
            place_id: 3,
            date: 1555816218,
            hoursStart: 1553045407,
            hoursEnd: 1553052607,
            name: 'Tình hình báo cáo nhân sự'
          },
          {
            place_id: 2,
            date: 1555816218,
            hoursStart: 1553045407,
            hoursEnd: 1553052607,
            name: 'Tình hình báo cáo nhân sự'
          },
          {
            place_id: 8,
            date: 1555816218,
            hoursStart: 1555837818,
            hoursEnd: 1555841928,
            name: 'Báo cáo tình hình phát triển trung tâm công nghệ thông tin Mobifone'
          }
        ];
      },
      type: Array
    },
    heightTable: {
      default: 450,
      type: Number
    },
    widthSessionOfTheDay: {
      default: 250,
      type: Number
    },
    loadingTable: {
      default: false,
      type: Boolean
    },
    customClassEventByDay: {
      default: '',
      type: String
    },
    customClassEventByWeek: {
      default: '',
      type: String
    }
  },
  computed: {
    dateCurrentFormatUTC: {
      get: function () {
        return this.dateCurrent
      }
    },
    listEvents: function () {
      return this.listEvent
    },
    // computed by week
    dayOfWeek: function () {
      return this.listDateofWeek
    }
  },
  watch: {
    date: function (newValue) {
      console.log(newValue)
      return (newValue && (this.dateCurrentFormatUTC = newValue))
    }
  },
  methods: {
    pad (num) {
      return ("0" + num).slice(-2);
    },

    getTimeFromDate (timestamp) {
      var date = new Date(timestamp * 1000);
      var minutes = date.getMinutes();
      var hours = date.getHours();
      return this.pad(hours) + ':' + this.pad(minutes);
    },
    getHoursFromDate (timestamp) {
      var date = new Date(timestamp * 1000);
      var hours = date.getHours();
      return this.pad(hours)
    },
    getMinutesFromDate (timestamp) {
      var date = new Date(timestamp * 1000);
      var minutes = date.getMinutes();
      return this.pad(minutes)
    },

    getDistanceToLeft (start) {
      let distanceHours = (this.getHoursFromDate(start) - 8) * 120
      let distanceMinutes = this.getMinutesFromDate(start) * 2
      return (distanceHours + distanceMinutes + 10)
    },

    getWidthAEvent (start, end) {
      let distanceHours = (this.getHoursFromDate(end) - this.getHoursFromDate(start)) * 120
      let distanceMinutes = (this.getMinutesFromDate(end) - this.getMinutesFromDate(start)) * 2
      return (distanceHours + distanceMinutes)
    },


    // by week

    getDateFormatFromDate (timestamp) {
      var date = new Date(timestamp * 1000)
      return date.toLocaleDateString(['ban', 'id']);
    },


    prevDay () {
      this.dateCurrent = moment(this.dateCurrent).add(-1, 'days')
      this.$emit('prevDay')
      this.$emit('loadData')
    },
    nextDay () {
      this.dateCurrent = moment(this.dateCurrent).add(1, 'days')
      this.$emit('nextDay')
      this.$emit('loadData')
    },

    getEvent (event) {
      this.$emit('eventClick', event)
    },


    prevWeek () {
      this.$emit('loadData')
      var pastDate = this.dateWeeken.getDate() - 7
      this.dateWeeken.setDate(pastDate)
      this.getDayofWeek(this.dateWeeken)
      this.$emit('prevWeek')
    },

    nextWeek () {
      this.$emit('loadData')
      var pastDate = this.dateWeeken.getDate() + 7
      this.dateWeeken.setDate(pastDate)
      this.getDayofWeek(this.dateWeeken)
      this.$emit('nextWeek')
    },


    getDayofWeek (date) {
      this.listDateofWeek = []
      var first = date.getDate() - 7;
      for (var i = 1; i < 8; i++) {
        var next = new Date(date.getTime());
        next.setDate(first + i);
        this.listDateofWeek.push(next.toLocaleDateString(['ban', 'id']));
      }
    },

    changeDate (value) {
      let timeSpecific = Math.floor(value.getTime() / 1000)
      this.$emit('loadData', timeSpecific)
    }
  },

  data () {
    return {
      dateCurrent: moment().utc()._d,
      dateWeeken: moment()._d,
      listDateofWeek: []
    }
  },

  created () {
    this.getDayofWeek(this.dateWeeken)
  },
}
</script>

<style scoped>
</style>
