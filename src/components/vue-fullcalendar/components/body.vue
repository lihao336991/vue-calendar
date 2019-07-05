<template>
  <div class="full-calendar-body">
    
    <div class="right-body">
      <div class="weeks">
        <div class="blank" v-if="tableType=='week'" style="width: 60px"></div>
        <strong class="week" v-for="(week,index) in weekNames">
          {{week}}
          <span v-if="tableType=='week' && weekDate.length">({{weekDate[index].showDate}})</span>
        </strong>
      </div>
      <div class="dates" ref="dates" v-if="tableType=='month'">
        <Spin v-if="loading" fix></Spin>
        <div class="dates-events">
          <div class="events-week" v-for="week in currentDates" v-if="week[0].isCurMonth || week[week.length-1].isCurMonth">
            <div class="events-day" v-for="day in week" track-by="$index"
              :class="{'today' : day.isToday,
                'not-cur-month' : !day.isCurMonth}">
              <p class="day-number">{{day.monthDay}}</p>
              <div class="event-box" v-if="day.events.length">
                <div class="event-item"
                :class="{selected: showCard == event.id}"
                 v-for="event in day.events"
                  :style="`background-color:${event.color ? event.color : '#C7E6FD'}`"
                  @click="eventClick(event,$event)">
                  <span :class="`icon icon${event.id%4}`">{{event.name}}</span>
                  <span class="info">{{isBegin(event, day.date, day.weekDay)}}</span>
                  <div class="card" v-if="event &&showCard == event.id" @click.stop>
                    <slot name="body-card"></slot>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <div class="time" ref="time" v-else-if="tableType=='week'">
        <Spin v-if="loading" fix></Spin>
        <div class="row" v-for="(time,index) in timeDivide">
          <div class="left-info" v-if="tableType=='week'">
            <div class="time-info first" v-if="index==0">
              <span class="center">上午</span>
            </div>
            <div class="time-info" v-if="index==1">
              <span class="top">12:00</span>
              <span class="center">下午</span>
            </div>
            <div class="time-info" v-if="index==2">
              <span class="top">18:00</span>
              <span class="center">晚上</span>
            </div>
          </div>
          <div  class="events-day" v-for="item in weekDate" v-if="weekDate.length" :class="{today: item.isToday}">
            <div class="event-box" v-if="item.events.length">
              <div class="event-item" v-for="event in item.events"
                :class="{selected: showCard == event.id}"
              :style="`background-color:${event.color ? event.color : '#C7E6FD'}`"
                v-if="isTheday(item.date, event.start) && isInTime(time, event.start)"
                @click="eventClick(event,$event)">
                <span :class="`icon icon${event.id%4}`">{{event.name}}</span>
                <span class="info">{{event.title}}</span>
                <div class="card" v-if="event && showCard == event.id" @click.stop>
                  <slot name="body-card"></slot>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script type="text/babel">
  import dateFunc from './dateFunc'
  import bus from './bus'

  export default {
    props : {
      currentDate : {},
      events      : {
        type: Array,
        default: []
      },
      weekNames   : {
        type : Array,
        default : []
      },
      monthNames  : {},
      firstDay    : {},
      tableType   : '',
      weekDays    : {
        type: Array,
        default () {
          return dateFunc.getDates(new Date())
        }
      },
      isLimit     : {
        type: Boolean,
        default   : false
      },
      loading: {
        type: Boolean,
        default : true
      }
    },
    created () {
      let _this = this
      document.addEventListener('click', function(e){
        _this.showCard = -1
      })
      // 监听header组件事件
      bus.$on('changeWeekDays', res=>{

      })
    },
    data () {
      return {
        // weekNames : DAY_NAMES,
        weekMask : [1,2,3,4,5,6,7],
        // events : [],
        eventLimit : 18,
        showMore : false,
        morePos : {
          top: 0,
          left : 0
        },
        selectDay : {},
        timeDivide : [{
          start: 0,
          end: 12,
          label: '上午'
        },{
          start: 12,
          end: 18,
          label: '下午'
        },{
          start: 18,
          end: 23,
          label: '晚上'
        }],
        showCard: -1,
      }
    },
    
    watch: {
      // events(val){
      //   this.getCalendar()
      // },
      currentDate(){
        this.getCalendar()
      }
    },
    computed : {
      currentDates () {
        return this.getCalendar()
      },
      weekDate () {
        return this.getWeekDate()
      }
    },
    methods : {
      isBegin (event, date, index) {
        let st = new Date(event.start)

        if (index == 0 || st.toDateString() == date.toDateString()) {
          return event.title
        }
        return '　'
      },
      moreTitle (date) {
        let dt = new Date(date)
        return this.weekNames[dt.getDay()] + ', ' + this.monthNames[dt.getMonth()] + dt.getDate()
      },
      classNames (cssClass) {
        if(!cssClass) return ''
        // string  
        if (typeof cssClass == 'string') return cssClass

        // Array
        if (Array.isArray(cssClass)) return cssClass.join(' ')
        
        // else
        return ''
      },
      getCalendar () {
        // calculate 2d-array of each month
        // first day of this month
        let now = new Date() // today
        let current = new Date(this.currentDate)

        let startDate = dateFunc.getStartDate(current) // 1st day of this month

        let curWeekDay = startDate.getDay()

        // begin date of this table may be some day of last month
        let diff = parseInt(this.firstDay) - curWeekDay + 1
        diff = diff > 0 ? (diff - 7) : diff

        startDate.setDate(startDate.getDate() + diff)
        let calendar = []
        for(let perWeek = 0 ; perWeek < 6 ; perWeek++) {

          let week = []

          for(let perDay = 0 ; perDay < 7 ; perDay++) {
            // console.log(startDate)
            week.push({
              monthDay : startDate.getDate(),
              isToday : now.toDateString() == startDate.toDateString(),
              isCurMonth : startDate.getMonth() == current.getMonth(),
              weekDay : perDay,
              date : new Date(startDate),
              events : this.slotEvents(new Date(startDate))
            })
            startDate.setDate(startDate.getDate() + 1)
          }
          calendar.push(week)
        }
        return calendar
      },
      slotEvents (date) {
        // console.log(date)
        let thisDayEvents = []
        this.events.filter(event => {
          let day = new Date(event.start)
          if(date.toLocaleDateString() === day.toLocaleDateString()){
            thisDayEvents.push(event)
          }
        })
        this.judgeTime(thisDayEvents)
        return thisDayEvents
      },
      // 获取周视图的天元素格式化
      getWeekDate(){
        let newWeekDays = this.weekDays
        newWeekDays.forEach((e,index) => {
          e.showDate = dateFunc.format(e, 'MM-dd');
          e.date = dateFunc.format(e, 'yyyy-MM-dd');
          e.isToday = (new Date().toDateString() == e.toDateString())
          e.events = this.slotTimeEvents(e) // 整理事件集合 （拿事件去比较时间，分发事件到时间插槽内）
        })
        return newWeekDays
      },
      // 发现该时间段所有的事件
      slotTimeEvents(date){
        let thisDayEvents = this.events
        thisDayEvents.filter(event => {
          let day = new Date(event.start)
          return date.toLocaleDateString() == day.toLocaleDateString()
        })
        this.judgeTime(thisDayEvents)
        return thisDayEvents
      },
      judgeTime(arr){
        arr.forEach(event => {
          let day = new Date(event.start)
          // 加上时间戳后判断时间段
          let hour = day.getHours()
          let week = day.getDay()
          if (this.timeDivide[0].start < hour < this.timeDivide[0].end) {
              event.time = 0
          } else if (this.timeDivide[1].start < hour < this.timeDivide[1].end) {
              event.time = 1
          } else if (this.timeDivide[2].start < hour < this.timeDivide[2].end) {
              event.time = 2
          }
          event.weekDay = this.weekNames[Number(week) - 1]
          event.weekDayIndex = week
        })
      },
      isTheday(day1,day2){
        return new Date(day1).toDateString() === new Date(day2).toDateString()
      },
      isStart (eventDate, date) {
        let st = new Date(eventDate)
        return st.toDateString() == date.toDateString()
      },
      isEnd (eventDate,date) {
        let ed = new Date(eventDate)
        return ed.toDateString() == date.toDateString()
      },
      isInTime(time, date){
        let hour = new Date(date).getHours()
        return (time.start <= hour) && (hour < time.end)
      },
      selectThisDay (day, jsEvent) {
        this.selectDay = day
        this.showMore = true
        this.morePos = this.computePos(event.target)
        this.morePos.top -= 100
        let events = day.events.filter(item =>{
          return item.isShow == true
        })
        this.$emit('moreclick', day.date, events, jsEvent)
      },
      computePos (target) {
        let eventRect = target.getBoundingClientRect()
        let pageRect = this.$refs.dates.getBoundingClientRect()
        return {
          left : eventRect.left - pageRect.left,
          top  : eventRect.top + eventRect.height - pageRect.top
        }
      },
      dayClick(day, jsEvent) {
        console.log('dayClick')
        this.$emit('dayclick', day, jsEvent)
      },
      eventClick(event, jsEvent) {
        console.log(event,jsEvent, 'evenvet')
        this.showCard = event.id
        jsEvent.stopPropagation()
        this.$emit('eventclick', event, jsEvent)
      },
    }
  }
</script>
<style lang="scss">
.full-calendar-body{
  background-color: #fff;
  display: flex;
  margin-top: 12px;
  border: 1px solid #D0D9FF;
  .left-info{
    width: 60px;
    .time-info{
      height: 100%;
      position: relative;
      &.first{
        border-top:  1px solid #EFF2FF;
      }
      &:nth-child(2){
        border-top: 1px solid #EFF2FF;
        border-bottom: 1px solid #EFF2FF;
      }
      .center{
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%,-50%);
        width: 14px; 
        font-size: 14px; 
        word-wrap: break-word; 
        letter-spacing: 10px; 
      }
      .top{
        position: absolute;
        top: -8px;
        left: 0;
        width: 100%;
        text-align: center;
      }
    }
  }
  .right-body{
    flex: 1;
    width: 100%;
    position: relative;
    .weeks{
      display: flex;
      border-bottom:1px solid #FFCC36;
      .week{
        flex:1;
        text-align: center;
        height: 40px;
        display: flex;
        align-items: center;
        justify-content: center;
      }
    }
    .dates {
      position:relative;
      // overflow: auto;
      // max-height: calc(100vh - 280px);
      .dates-events{
        z-index:1;
        width: 100%;
        .events-week{
          display: flex;
          min-height: 180px;
          .events-day{
            // cursor: pointer;
            text-overflow: ellipsis;
            flex:1;
            width: 0;
            height: auto;
            padding:4px;
            border-right:1px solid #EFF2FF;
            border-bottom:1px solid #EFF2FF;
            background-color: #fff;
            .day-number{
              text-align: left;
              padding:4px 5px 4px 4px;
            }
            &.not-cur-month{
              .day-number{
                color:#8B8F94;
              }
            }

            &.today{
              background-color:#FFFCF3;
            }
            &:last-child{
              border-right: 0;
            }
            .event-box{
              .event-item{
                cursor: pointer;
                font-size:12px;
                background-color:#C7E6FD;
                margin-bottom:2px;
                color: rgba(0,0,0,.87);
                padding:0 0 0 4px;
                height: 30px;
                line-height: 30px;
                position: relative;
                display: flex;
                align-items: center;
                border-radius: 4px;
                &.is-end{
                  display: none;
                }
                &.is-start{
                  display: block;
                }
                &.is-opacity{
                  display: none;
                }
                .icon{
                  width: 20px;
                  height: 20px;
                  line-height: 20px;
                  border-radius: 10px;
                  text-align:center;
                  color: #fff;
                  display: inline-block;
                  &.icon0{
                    background: #27BA9C;
                  }
                  &.icon1{
                    background: #5272FF;
                  }
                  &.icon2{
                    background: #FFCC36;
                  }
                  &.icon3{
                    background: #FF7062;
                  }
                }
                .info{
                  width: calc(100% - 30px);
                  overflow: hidden;
                  white-space: nowrap;
                  text-overflow: ellipsis;
                  display: inline-block;
                  margin-left: 5px;
                }
                .card{
                  cursor: initial;
                  position: absolute;
                  z-index: 999;
                  top: 40px;
                  left: calc(50% - 125px);
                  width: 250px;
                  height: auto;
                  min-height: 100px;
                  background: #fff;
                  // border: 1px solid #eee;
                  box-shadow:0px 2px 10px 0px rgba(0,0,0,0.1);
                  border-radius: 4px;
                  overflow: hidden;
                }
                &.selected{
                  .info{
                    color: #fff;
                    font-weight: normal;
                  }
                  background-color: #5272FF !important;
                  .icon{
                    background-color: #5272FF !important;
                  }
                }
              }
              .more-link{
                cursor: pointer;
                // text-align: right;
                padding-left: 8px;
                padding-right: 2px;
                color: rgba(0,0,0,.38);
                font-size: 12px;
              }
            }
          }
          &:last-child{
            .events-day{
              border-bottom: 0;
            }
          }
        }
      }
      .more-events{
        position:absolute;
        width: 150px;
        z-index: 2;
        border:1px solid #eee;
        box-shadow: 0 2px 6px rgba(0,0,0,.15);
        .more-header{
          background-color:#eee;
          padding:5px;
          display: flex;
          align-items : center;
          font-size: 14px;
          .title{
            flex:1;
          }
          .close{
            margin-right: 2px;
            cursor: pointer;
            font-size: 16px;
          }
        }
        .more-body{
          height: 125px;
          overflow: hidden;
          background: #fff;
          .body-list{
            height: 120px;
            padding:5px;
            overflow: auto;
            background-color:#fff;
            .body-item{
              cursor: pointer;
              font-size:12px;
              background-color:#C7E6FD;
              margin-bottom:2px;
              color: rgba(0,0,0,.87);
              padding:0 0 0 4px;
              height: 18px;
              line-height: 18px;
              white-space: nowrap;
              overflow: hidden;
              text-overflow: ellipsis;
            }
          }
        }
      }
    }
    .time{
      position: relative;
      .row{
        width: 100%;
        display: flex;
        min-height: 180px;
        .events-day{
          border-bottom: 1px solid #EFF2FF;
          border-left: 1px solid #EFF2FF;
          background-color: #fff;
          height: auto;
          text-overflow: ellipsis;
          flex:1;
          width: 0;
          padding:4px;
          &.today{
            background-color: #FFFCF3;
          }
        }
        .event-item{
          cursor: pointer;
          font-size:12px;
          background-color:#C7E6FD;
          margin-bottom:2px;
          color: rgba(0,0,0,.87);
          padding:0 0 0 4px;
          height: 30px;
          line-height: 30px;
          white-space: nowrap;
          text-overflow: ellipsis;
          display: flex;
          align-items: center;
          position: relative;
          border-radius: 4px;
          .icon{
            width: 20px;
            height: 20px;
            line-height: 20px;
            border-radius: 10px;
            text-align:center;
            color: #fff;
            display: inline-block;
            &.icon0{
              background: #27BA9C;
            }
            &.icon1{
              background: #5272FF;
            }
            &.icon2{
              background: #FFCC36;
            }
            &.icon3{
              background: #FF7062;
            }
          }
          .info{
            width: calc(100% - 30px);
            overflow: hidden;
            white-space: nowrap;
            text-overflow: ellipsis;
            display: inline-block;
            margin-left: 5px;
          }
          .card{
            cursor: initial;
            position: absolute;
            z-index: 999;
            top: 40px;
            left: calc(50% - 125px);
            width: 250px;
            height: auto;
            min-height: 100px;
            background: #fff;
            box-shadow:0px 2px 10px 0px rgba(0,0,0,0.1);
            border-radius: 4px;
            overflow: hidden;
          }
          &.selected{
            .info{
              color: #fff;
              font-weight: normal;
            }
            background-color: #5272FF !important;
            .icon{
              background-color: #5272FF !important;
            }
          }
        }
        &:last-child{
          .single{
            border-bottom: 0;
          }
        }
      }
    }
  }
}
</style>
