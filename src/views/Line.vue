<template>
  <div style="overflow-x:hidden;height: 100%">
    <div class="panel">
      <div>
        <h4 style="float: left"><span class="glyphicon glyphicon-calendar" style="padding-top: 4px;padding-right: 5px"></span>路线列表</h4>
        <router-link class="btn btn-default" style="float:right;margin-top:10px;margin-right: 10px" to="lineSet">添加路线</router-link>
      </div>
    </div>
    <br>
    <div class="panel-body" style="height: 100%">
      <div>
        <input type="text" placeholder="输入关键字进行搜索" style="width: 400px;float: left;height: 34px" v-model="keywords">
      </div>
      <br>
      <br>
      <div class="list">
        <div class="table-bordered" style="background-color: white">
        <table class="table table-hover table-striped" width='100%' border='0' cellspacing='0' cellpadding='0' style='table-layout: fixed'>
          <thead>
          <tr>
            <th align="left" style="width: 300px">景点名称</th>
            <th align="left">日程</th>
            <th align="left">票种</th>
            <th align="left">价格</th>
            <th align="left">余票</th>
            <th align="left">操作</th>
          </tr>
          </thead>
          <tbody>
          <!--<tr v-for="(item,index) in search(keywords).slice(x,y)">-->
          <tr v-if="line.length!==0" v-for="(item,index) in line">
          <td>{{item.name}}</td>
            <td v-if="item.tripItems[0]">{{item.tripItems[0].useTime}}</td>
            <td v-else>未定</td>
            <td>{{item.ticketstyle}}</td>
            <td>{{item.ticketprice}}</td>
            <td>{{item.ticketleave}}</td>
            <td>
              <router-link to="lineSet"><a v-on:click="getIndex(line[index])">修改</a></router-link>/<a href="#" v-on:click="deleteInfo(index)">删除</a>
            </td>
          </tr>
          </tbody>
        </table>
      </div>
      </div>

      <nav aria-label="...">
        <ul class="pager">
          <li class="previous" v-on:click="shang" v-if="page!=1"><a href="#"><span aria-hidden="true">&larr;</span> 上一页</a></li>
          <li class="previous disabled" v-if="page==1"><a href="#"><span aria-hidden="true">&larr;</span> 上一页</a></li>
          <li class="next" v-on:click="xia"><a href="#">下一页<span aria-hidden="true">&rarr;</span></a></li>
        </ul>
      </nav>

    </div>
  </div>
</template>

<script>
  export default {
    data(){
      return{
        keywords:'',
        page:'1',
        num:'0',
        x:'0',
        y:'10',
        dataIndex:'',
        line:null
      }
    },
    created () {
      this.$axios.get(this.GLOBAL.BASE_URL+'/api/trip')
        .then(response => {
          this.line = response.data;
        })
    },
    methods:{
      deleteInfo(index){
        this.$axios.delete(this.GLOBAL.BASE_URL+'/api/trip/'+this.line[index].id)
        this.line.splice(index,1)
      },
      shang(){
        this.page--;
        this.$axios.get(this.GLOBAL.BASE_URL+'/api/trip',{
          params:{
            page: this.page
          }
        })
          .then(response => {
            if (!(Array.isArray(response.data))||response.data==null||response.data.length===0){
              this.page++;
              return;
            }
            this.line = response.data;
          })
      },
      xia(){
        this.page++;
        this.$axios.get(this.GLOBAL.BASE_URL+'/api/trip',{
          params:{
            page: this.page
          }
        })
          .then(response => {
            if (!(Array.isArray(response.data))||response.data==null||response.data.length===0){
              this.page--;
              return;
            }
            this.line = response.data;
          })
      },
      getIndex(line){
        console.log(this.line)
        this.$router.push({
          name:'lineset',
          params:{
            line : line
          }
        })
      },
      search() {
        this.$axios.get(this.GLOBAL.BASE_URL+'/api/trip', {
          params:{
            fuzzyKey: this.keywords,
            page: this.page
          }
        }).then(response => this.line = response.data)
      }
    },
    filters:{
      dateFormat:function(time) {
        var date=new Date(time);
        var year=date.getFullYear();
        var month= date.getMonth()+1<10 ? "0"+(date.getMonth()+1) : date.getMonth()+1;
        var day=date.getDate()<10 ? "0"+date.getDate() : date.getDate();
        var hours=date.getHours()<10 ? "0"+date.getHours() : date.getHours();
        var minutes=date.getMinutes()<10 ? "0"+date.getMinutes() : date.getMinutes();
        var seconds=date.getSeconds()<10 ? "0"+date.getSeconds() : date.getSeconds();
        // 拼接
        return year+"-"+month+"-"+day+" "+hours+":"+minutes+":"+seconds;
      }
    },
    watch:{
      keywords(key){
        this.search()
      }
    }
  }
</script>

<style>
  .list{
    width: fit-content;
  }
  .table{
    margin-bottom: 0px;
  }
</style>
