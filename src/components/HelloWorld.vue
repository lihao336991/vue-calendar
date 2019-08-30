<template>
  <div>
    <full-calendar :events="fcEvents" @eventClick="fetchEvent" lang="zh" @change="changeDateRange">
      <template slot="fc-body-card">
          <div class="card-content"  v-if="interviewEvent">
              <p class="card-info">{{interviewEvent.weekDay}}{{interviewEvent.title}}</p>
              <p class="card-name">{{interviewEvent.name}}</p>
              <!-- <p class="card-info">手机：{{interviewEvent.data.cv.contact.mobile}}</p> -->
              <p class="card-info">id:{{interviewEvent.id}}</p>
              <div class="card-handle">
                  <div class="card-left">
                      请假1天
                  </div>
                  <span class="card-right link">查看详情</span>
              </div>
              <div class="card-handle">
                  <span class="card-left">审核人：{{'某某'||interviewEvent}}</span>
                  <span class="card-right status status1">已通过</span>
              </div>
          </div>
      </template>
  </full-calendar>
  </div>
</template>
<script>
import FullCalendar from './vue-fullcalendar/fullCalendar.vue'
import mock from './mock.json'
export default {
  components:{
    "full-calendar": FullCalendar
  },
  data(){
    return{
      fcEvents: [],
      interviewEvent: {},
    }
  },
  created(){
    this.fetchEventsList();
  },
  methods:{
    fetchEvent(event, jsEvent){
      this.interviewEvent = event
      console.log(event, jsEvent)
    },
    changeDateRange(){
      console.log('改变了日期')
    },
    fetchEventsList(){
      this.initEvents(mock)
    },
    initEvents(list){
      this.fcEvents = mock
      
      console.log(this.fcEvents, 'fc')   
    }
  }
}
</script>
<style scoped lang="scss">

  .card-content{
      box-sizing: border-box;
      width: 100%;
      height: auto;
      overflow: hidden;
      position: relative;
      padding: 5px 10px;
      text-align: left;
      .card-name{
          font-weight: bold;
          font-size: 14px;
          color: #3E444C;
      }
      .card-info{
          font-size: 13px;
          color: #3E444C;
          text-overflow: ellipsis;
          width: 100%;
          overflow: hidden;
          word-wrap: none;
      }
      .card-handle{
          font-size: 13px;
          color: #3E444C;
          display: flex;
          align-items: center;
          justify-content: space-between;
          &:last-child{
              border-top: 1px solid #ECECED;
              padding-top: 5px;
              margin-top: 5px;
          }
          .card-left{
              display: flex;
              align-items: center;
              .tag{
                  display: inline-block;
                  width: 50px;
                  height: 20px;
                  margin-left: 10px;
              }
          }
          .card-right{
              &.status{
                  border-radius: 15px;
                  border: 1px solid;
                  padding: 0px 5px;
                  font-size: 12px;
                  margin: 0 4px;
                  word-break: keep-all;
                  white-space: nowrap;
                  display: inline-block;
                  line-height: 1.5;
                  &.status0{
                      color: #8B8F94;
                  }
                  &.status1{
                      color: #27BA9C;
                  }
                  &.status5{
                      color: #FF7062;
                  }
              }
              &.link{
                  color: #5272FF;
                  font-size: 11px;
                  cursor: pointer;
                  &:hover{
                      color: #627FFF;
                  }
              }
          }
      }
  }
</style>
