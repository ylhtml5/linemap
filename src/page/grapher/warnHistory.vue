<template>
  	<div class="warnHistory">

       <div class="search">
          <el-row>

            <el-col :span="7"><div class="grid-content bg-purple-light">
                <span>所属分组：</span><el-input class="serinput" v-model="searchData.group" @change="search" placeholder="请输入分组"></el-input>
            </div></el-col>

            <el-col :span="7"><div class="grid-content bg-purple-light">
                <span>IMEI：</span><el-input class="serinput" v-model="searchData.imei" @change="search" placeholder="请输入IMEI"></el-input>
            </div></el-col>

            <el-col :span="8"><div class="grid-content bg-purple-light">
                <span>时间段：</span>
                <el-date-picker
                      v-model="dateM"
                      class="serinput"
                      type="datetimerange"
                      placeholder="选择日期范围"
                      @change='filterSearch'>
                    </el-date-picker>
            </div></el-col>
  
          </el-row>

          <el-row>

            <el-col :span="7"><div class="grid-content bg-purple-light">
                <span>地址位置：</span><el-input class="serinput" v-model="searchData.address" @change="search" placeholder="请输入地址位置"></el-input>
            </div></el-col>

          </el-row>
       </div>

      <el-table
        :data="userData"
        style="width: 96%;margin-left:2%;"
        border
        >
        <el-table-column
          prop="imei"
          align='center'
          label="IMEI">
        </el-table-column>
        <el-table-column
          prop="group"
          align='center'
          label="所属分组">
        </el-table-column>
        <el-table-column
          show-overflow-tooltip
          align='center'
          label="车牌号">

          <template scope="scope">
                {{ }}
          </template>

        </el-table-column>
        <el-table-column
          prop="address"
          align='center'
          label="地址信息">
        </el-table-column>
        <el-table-column
          align='center'
          label="时间">

           <template scope="scope">
              {{scope.row.timeUnix|filterTime}}
          </template>

        </el-table-column>
      </el-table>

      <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
      </el-pagination>

    </div>
</template>

<script>

import {warnHisList} from '../../service/getData'
import {layer, exit, checkPro, DateFormat} from '../../components/common/common'
import _ from 'lodash'

export default {
    data(){
        return{
          searchData:{
            group: '',
            imei: '',
            startTime: '',
            endTime: '',
            address: ''
          },
          dateM: [],
          userData:[
          ],
          pageSize:10,
          total:0,
          currentPage:1,
          start:0,
          count:10,
          page:1,
        }
    },

	  mounted(){
       exit(this)
       checkPro('admin', this, true)
       this.initUser()         
    },

    components:{

    },

    computed:{
        
    },

    watch:{
      dateM(val, oldval){
        if (val instanceof Array
          && val.length > 1) {
          this.searchData.startTime = val[0]&&val[0].getTime()
          this.searchData.endTime = val[1]&&val[1].getTime()
        }
      },
    },

    methods:{

      search:_.debounce(function(){
          this.filterSearch()
      }, 400),

      handleSizeChange(size){
        this.pageSize = size;
        this.count = size;
        this.initUser((this.page-1)*size, size, this.page)
      },

      handleCurrentChange(currentPage){
        this.currentPage = currentPage;
        this.page = currentPage;
        this.initUser((parseInt(currentPage)-1)*this.count, this.count, currentPage)
      },

      async initUser(start = 0, count = 10, page = 1, ) {

          let obj = {
              start : start,
              count : count,
          };

          for(let key in this.searchData){
              
              if(this.searchData[key]){
                  obj[key] = this.searchData[key] 
              }

          }
          
          let data = await warnHisList('get', obj);
          if(data.code == 0){
              this.total = data.res.totalCount;
              this.userData = data.res.events;
              this.start = start;
              this.count = count;
              this.page = page;
          }else{
              layer(this,data.ch||data.message||"请求错误")
          }
      },

      filterSearch(){
        this.currentPage = 1
        this.initUser(0,this.count,1)
      },

    },

    filters:{
      filterTime(val){
          Date.prototype.format = DateFormat;
          let Time  = new Date(val).format('yyyy-MM-dd hh:mm:ss')
          return Time;
      },
    }
}

</script>

<style lang="scss" scoped>
    @import '../../style/mixin';
    .warnHistory{
        position: absolute;
        font-size: 14px;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        height: 100%;
        width: 100%;
        padding-bottom: .16rem;
        .search{
          width: 96%;
          margin-left:2%;
          margin-top: .1rem;
          margin-bottom: .3rem;
          .el-row:nth-child(1){
            margin-bottom: .18rem;
          }
          .serinput{
            width: 70%;
          }
          .addBt{
            float:right;
            margin-right: 3%;
          }
          .el-icon-plus{
            font-size: 12px;
            margin-right: .03rem;
          }
        }
        .el-pagination{
          text-align: center;
          margin-top: .2rem;
        }
    }

</style>
