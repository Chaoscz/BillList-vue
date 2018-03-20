<template>
    <div class="today">
      <el-date-picker
        v-model="pickerValue1"
        align="right"
        type="date"
        placeholder="选择日期"
        :picker-options="pickerOptions">
      </el-date-picker>

      <el-button type="primary" icon="search">查询</el-button>
      <el-button type="primary" icon="plus" @click="increase()">添加</el-button>

      <!-- 账单表 -->
      <div class="billtable">
      <el-table :data="tableData"   stripe fit>

      <el-table-column  label="金额" >
        <template scope="scope">
            <span >{{ scope.row.billMoney }}</span>
        </template>
      </el-table-column>

      <el-table-column label="账单类别" >
        <template scope="scope">
            <span style="margin-left: 10px">{{ scope.row.billType }}</span>
        </template>
      </el-table-column>

      <el-table-column label="消费类别" >
        <template scope="scope">
            <span style="margin-left: 10px">{{ scope.row.consumptionType }}</span>
        </template>
      </el-table-column>

      <el-table-column label="时间" >
        <template scope="scope">
            <span style="margin-left: 10px">{{ scope.row.consumptionType }}</span>
        </template>
      </el-table-column>

      <el-table-column label="地点" >
        <template scope="scope">
            <span style="margin-left: 10px">{{ scope.row.consumptionSpace }}</span>
        </template>
      </el-table-column>

      <el-table-column label="人物" >
        <template scope="scope">
            <span style="margin-left: 10px">{{ scope.row.hasWho }}</span>
        </template>
      </el-table-column>

      <el-table-column label="备注" >
        <template scope="scope">
            <span style="margin-left: 10px">{{ scope.row.remark }}</span>
        </template>
      </el-table-column>
      
          <el-table-column label="操作">
            <template scope="scope">
             <!--  <el-button size="small"  @click="handleEdit(scope.$index, scope.row)">修改</el-button> -->
              <el-button size="small" type="danger" @click="handleDelete(scope.$index)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <div>
        <el-dialog title="消费记录" :visible.sync="dialogFormVisible">
          <el-form :model="form">
            <el-form-item label="日期" :label-width="formLabelWidth">
              <el-date-picker type="date" placeholder="选择日期" v-model="pickerValue2"></el-date-picker>
            </el-form-item>
            <el-form-item label="金额" :label-width="formLabelWidth">
              <el-input v-model="newdata.billMoney" auto-complete="off"></el-input>
            </el-form-item>
            <el-form-item label="备注" :label-width="formLabelWidth">
              <el-input v-model="newdata.remark" auto-complete="off"></el-input>
            </el-form-item>

            <el-form-item label="账单类别" :label-width="formLabelWidth">
              <el-input v-model="newdata.billType" auto-complete="off"></el-input>
            </el-form-item>

            <el-form-item label="消费类别" :label-width="formLabelWidth">
              <el-input v-model="newdata.consumptionType" auto-complete="off"></el-input>
            </el-form-item>
      
             <el-form-item label="地点" :label-width="formLabelWidth">
              <el-input v-model="newdata.consumptionSpace" auto-complete="off"></el-input>
            </el-form-item>

            <el-form-item label="和谁" :label-width="formLabelWidth">
              <el-input v-model="newdata.hasWho" auto-complete="off"></el-input>
            </el-form-item>
            
            <el-form-item style="text-align: center">
            <el-button @click="dialogFormVisible = false">取 消</el-button>
            <el-button type="primary" @click="addbill">确 定</el-button>
            </el-form-item>
          </el-form>
          
        </el-dialog>
      </div>

        
    </div>
</template>

<script>
    import querytime from '../public/time'
    export default {
        name: '',
        data: function () {
            return {       
              tableData: [],
              dialogTableVisible: false,
              dialogFormVisible: false,
              newdata:{
                userId: 2,
                billMoney: '',
                billType: '',
                consumptionType: '',
                consumptionSpace: '',
                hasWho: "",
                remark: ""
              },
              
              form: {
                namedate: '',
                region: '',
                date1: '',
                date2: '',
                delivery: false,
                type: [],
                resource: '',
                desc: '',
                remarks:'',
                time:'',
                money:''
              },
              formLabelWidth: '120px',
              boxdata:{},
              pickerOptions: {
              },
              pickerValue1: '',
              pickerValue2: '',
            }
        },
        components: {

        },
        created:function(){
          let limit = "";
          let id = "";
          this.$axios.get('http://localhost:8000/v1/bill/query?limit='+limit+'&id='+id+'').then((res) => {
          this.tableData = res.data.data
          })
        },
        methods:{
          addbill: function () {  //确定
            // const _this=this;
            // console.log("选择器中的时间：",_this.pickerValue2);
            // _this.newdata.date = _this.formatdate(_this.pickerValue2);

            // if(_this.newdata.id=="id"){
            //   console.log("修改数据状态" );
            //   _this.newdata= {date: _this.newdata.date, time: '', money: '', mdetails: '' }
            // }else{
            //   console.log("当index为空时，这时弹框就是添加数据状态");
            //   _this.tableData.push(_this.newdata);
            // }
            // console.log(_this.newdata);
            // _this.newdata="";
            // _this.dialogFormVisible = false;

            const _this = this;
            _this.tableData.push(_this.newdata);

            var dataArrary = {
              userId:_this.newdata.userId,
              billMoney: _this.newdata.billMoney,
              billType: _this.newdata.billType,
              consumptionType:_this.newdata.consumptionType,
              consumptionSpace:_this.newdata.consumptionSpace,              
              hasWho:_this.newdata.hasWho,  
              remark:_this.newdata.remark
            }

            _this.$axios.post('http://localhost:8000/v1/bill/save',dataArrary).then((res)=>{
              console.log(res)
            });

            _this.newdata="";
            _this.dialogFormVisible = false;

          },
          handleDelete:function (index) {  //删除
            // 后台删除
            var _this = this
            
            console.log(this.tableData)

            this.tableData.splice(index, 1);

          },
          //编辑数据
          handleEdit: function (index, row) { 
            this.newdata = row;
            this.pickerValue2 = row.date
            this.newdata.id =index;
            this.dialogFormVisible = true;
            console.log(this.newdata.id);
          },
          formatdate: function (mydate) {   //时间显示
            if(mydate){
              console.log("有时间数据");
              var reg = /^[1-9]\d{3}-(0[1-9]|1[0-2])-(0[1-9]|[1-2][0-9]|3[0-1])$/;
              var regExp = new RegExp(reg);
              if(!regExp.test(mydate)){
                console.log("没有通过正则")
                var y = mydate.getFullYear();
                var m = mydate.getMonth() + 1;
                m = m < 10 ? '0' + m : m;
                var d = mydate.getDate();
                d = d < 10 ? ('0' + d) : d;
                return y + '-' + m + '-' + d;
              }else{
                console.log("通过正则");
                return mydate;
              }

            }else{
              console.log("没有时间数据");
              return ''
            }
          },
          increase:function(){  //添加
            // this.newdata="";
            this.dialogFormVisible = true;
          }
        },
        filters: {
        },
        computed: {
           //computed可以关联一个变量  有get和set方法2.
        }
    }
</script>

<style scoped>

</style>
