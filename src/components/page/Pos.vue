<template>
  <div class="pos">
    <el-row>
      <el-col :span="7" class="pos-order" id="order-list">
        <el-tabs type="border-card">
          <el-tab-pane label="点餐">
            <el-table :data="tableData" border style="width: 100%;" >
              <el-table-column prop="gname" label="商品"  ></el-table-column>
              <el-table-column prop="count" label="数量" width="50"></el-table-column>
              <el-table-column prop="gprice" label="金额" width="70"></el-table-column>
              <el-table-column  label="操作" width="100" fixed="right">
                <template slot-scope="scope">
                  <el-button type="text" size="small" @click="delSingleGoods(scope.row)">删除</el-button>
                  <el-button type="text" size="small" @click="addOrderList(scope.row)">增加</el-button>
                </template>
              </el-table-column>
            </el-table>
            <div class="totalDiv">
              <small>数量 ：</small>{{totalCount}} &nbsp;&nbsp; <small>金额 ：</small>{{totalMoney}}元
            </div>
            <div class="order-btn">
              <el-button type="warning">挂单</el-button>
              <el-button type="danger" @click="delAllGoods">删除</el-button>
              <el-button type="success" @click="checkout">结账</el-button>
            </div>
          </el-tab-pane>
          <el-tab-pane label="挂单">
            挂单
          </el-tab-pane>
          <el-tab-pane label="外卖">
            外卖
          </el-tab-pane>
        </el-tabs>
      </el-col>
      <el-col :span="17">
        <div class="often-goods">
          <div class="title">常用商品</div>
          <div class="often-goods-list">
            <ul>
              <li v-for="goods in oftenGoods" @click="addOrderList(goods)">
                <span>{{goods.gname}}</span>
                <span class="price">￥{{goods.gprice}}元</span>
              </li>
            </ul>
          </div>
        </div>

        <div class="goods-type">
          <el-tabs type="border-card">
              <el-tab-pane label="汉堡">
                  <div>
                    <ul class="cookList">
                      <li v-for="goods in type0Goods" @click="addOrderList(goods)">
                        <span class="foodImg"><img :src="goods.gimg" width="100%"></span>
                        <span class="foodName">{{goods.gname}}</span>
                        <span class="foodPrice">￥{{goods.gprice}}元</span>
                      </li>
                    </ul>
                  </div>
              </el-tab-pane>
              <el-tab-pane label="小食">
                  <div>
                    <ul class="cookList">
                      <li @click="addOrderList(goods)" v-for="goods in type1Goods">
                        <span class="foodImg"><img :src="goods.gimg" width="100%"></span>
                        <span class="foodName">{{goods.gname}}</span>
                        <span class="foodPrice">￥{{goods.gprice}}元</span>
                      </li>
                    </ul>
                  </div>
              </el-tab-pane>
              <el-tab-pane label="饮料">
                  <div>
                    <ul class="cookList">
                      <li @click="addOrderList(goods)" v-for="goods in type2Goods">
                        <span class="foodImg"><img :src="goods.gimg" width="100%"></span>
                        <span class="foodName">{{goods.gname}}</span>
                        <span class="foodPrice">￥{{goods.gprice}}元</span>
                      </li>
                    </ul>
                  </div>
              </el-tab-pane>
              <el-tab-pane label="套餐">
                  <div>
                    <ul class="cookList">
                      <li @click="addOrderList(goods)" v-for="goods in type3Goods">
                        <span class="foodImg"><img :src="goods.gimg" width="100%"></span>
                        <span class="foodName">{{goods.gname}}</span>
                        <span class="foodPrice">￥{{goods.gprice}}元</span>
                      </li>
                    </ul>
                  </div>
              </el-tab-pane>
          </el-tabs>
        </div>
      </el-col>
    </el-row>
  </div>
</template>

<script>
import axios from 'axios';
export default {
  name: 'pos',
  data(){
    return {
      tableData: [],
      oftenGoods:[],
      type0Goods:[],
      type1Goods:[],
      type2Goods:[],
      type3Goods:[],
      totalMoney:0,
      totalCount:0
    }
  },
  created:function(){
    axios.get("http://localhost/php/oftenGoods.php")
    .then(response=>{
      this.oftenGoods=response.data;
    })
    .catch(error=>{
      alert("网络错误，不能访问")
    })

    axios.get("http://localhost/php/typeGoods.php")
    .then(response=>{
      this.type0Goods=this.type0Goods.concat(response.data[0],response.data[1],response.data[2],response.data[3]);
      this.type1Goods=this.type1Goods.concat(response.data[4],response.data[5]);
      this.type2Goods=this.type2Goods.concat(response.data[6],response.data[7]);
      this.type3Goods=this.type3Goods.concat(response.data[8],response.data[9]);
    })
    .catch(error=>{
      alert("网络错误，不能访问")
    })
  },
  mounted:function(){
    var orderHeight=document.body.clientHeight;
    console.log(orderHeight);
    document.getElementById('order-list').style.height=orderHeight+"px";
  },
  methods:{
    addOrderList(goods){
      // console.log(goods);
      this.totalMoney=0;
      this.totalCount=0;


      //判断商品是否存在订单列表
      let isHave=false;
      for(let i=0;i<this.tableData.length;i++){
        if(this.tableData[i].gid==goods.gid){
          isHave=true;
        }
      }
      //根据判断进行编写业务逻辑
      if(isHave){
        //改变列表中商品数量
        let arr=this.tableData.filter(o=>o.gid==goods.gid);
        arr[0].count++;
      }else{
        let newsGoods={gid:goods.gid,gname:goods.gname,gprice:goods.gprice,count:1};
        this.tableData.push(newsGoods);
      }
      //计算汇总金额和数量
      this.getAllMoney();
    },
    //删除单个商品
    delSingleGoods(goods){
      this.tableData=this.tableData.filter(o=>o.gid != goods.gid);
      this.getAllMoney();
    },
    //删除所有商品
    delAllGoods(){
      this.tableData=[];
      this.totalCount=0;
      this.totalMoney=0;
    },
    //结账
    checkout(){
      if(this.totalCount!=0){
        this.tableData=[];
        this.totalCount=0;
        this.totalMoney=0;
        //Element组件message
        this.$message({
          message:'结账成功，感谢你为店里出了一份力!',
          type:"success"
        });
      }else{
        this.$message.error("不能空结，老板了解你急切的心情!");
      }
    },
    //汇总数量金额
    getAllMoney(){
      this.totalCount=0;
      this.totalMoney=0;
      this.tableData.forEach(element=>{
        this.totalCount+=element.count;
        this.totalMoney=this.totalMoney+(element.gprice*element.count);
      })
    }
  }
}
</script>

<style scoped>

  .pos-order{
    background: #F9FAFC;
    border-right: 1px solid #C0CCDA;
  }
  .order-btn{
    margin-top: 10px;
    text-align: center;
  }
  .title{
    height:19px;
    border-bottom: 1px solid #D3dce6;
    background: #F9FAFC;
    padding:10px;
    text-align: left;
    font-size: 14px;
  }
  .often-goods ul li{
    list-style: none;
    float: left;
    border: 1px solid #F9FAFC;
    padding:10px;
    margin:5px;
    background: #FFF;
    cursor: pointer;
  }
  .price{
    color:#5887FF;
  }
  .goods-type{
    clear: both;
    position:relative;
    top:15px;
  }
  .cookList li{
      list-style: none;
      width:23%;
      border:1px solid #E5E9F2;
      height: auto;
      overflow: hidden;
      background-color:#fff;
      padding: 2px;
      float:left;
      margin: 2px;
      cursor: pointer;
  }
  .cookList li span{
      
      display: block;
      float:left;
  }
  .foodImg{
      width: 40%;
  }
  .foodName{
      font-size: 18px;
      padding-left: 10px;
      color:brown;

  }
  .foodPrice{
      font-size: 16px;
      padding-left: 10px;
      padding-top:10px;
  }
  .totalDiv {
    height: auto;
    overflow: hidden;
    text-align: right;
    font-size: 16px;
    background-color: #fff;
    border-bottom: 1px solid #E5E9F2;
    padding: 10px;
  }
</style>
