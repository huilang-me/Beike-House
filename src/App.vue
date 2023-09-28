<template>
  <div v-loading="loading">
    <meta name="referrer" content="never">
    <div v-if="showGetJson" class="get">
      <el-select allow-create v-model="houseId" filterable placeholder="房屋id" @change="houseIdChange">
        <el-option key="%252Fl3c2113333184394102" value="%252Fl3c2113333184394102" label="万科幸福誉"></el-option>
        <el-option key="%252Fpanyuguangchang%252Fbp150ep400" value="%252Fpanyuguangchang%252Fbp150ep400" label="番禺广场"></el-option>
        <el-option key="%2Fc2113327812276997" value="%2Fc2113327812276997" label="时代天韵"></el-option>
        <el-option key="%2Fc2120077370349227" value="%2Fc2120077370349227" label="绿地城"></el-option>
      </el-select>
      <el-input-number v-model="housePage" :min="1" :max="10" label="页码"></el-input-number>
      <el-switch v-model="getAll"></el-switch>
      <el-button type="primary" round @click="getJson">获取当前数据</el-button>
      <el-button class="copy" type="primary" round :data-clipboard-text="JSON.stringify(copyJson)">复制json</el-button>
    </div>
    <div class="compare">
      <el-select v-model="area" placeholder="区域" @change="areaChange">
        <el-option label="知识城" value="zhishicheng"/>
        <el-option label="番禺广场" value="panyuguangchang"/>
      </el-select>
      <el-select id="select" filterable clearable v-model="file" placeholder="选择旧数据" @change="fileChange">
        <el-option v-for="file in files" :key="file" :label="file" :value="file" />
      </el-select>
      <el-select id="select" filterable clearable v-model="file2" placeholder="选择新数据">
        <el-option v-for="file in files" :key="file" :label="file" :value="file" />
      </el-select>
      <el-button type="primary" round @click="compare">对比</el-button>
    </div>
    <el-table :data="compareResult.length==0 ?table.filter(data => (!search || data.desc.includes(search)) && (!tag || data.tags.includes(tag))):compareResult.filter(data => (!search || data.desc.includes(search)) && (!tag || data.tags.includes(tag)))" stripe style="width: 100%" :default-sort="{ prop: 'total', order: 'ascending' }">
      <el-table-column prop="id" sortable label="编号" width="70" />
      <el-table-column label="图片" width="60">
        <template #default="scope">
          <a :href="scope.row.link" target="_blank"><img :src="scope.row.img" width="60"></a>
        </template>
      </el-table-column>
      <el-table-column prop="title" label="标题"/>
      <el-table-column>
        <template #header>
          <el-select v-model="search" placeholder="搜索" filterable clearable allow-create>
            <el-option v-for="item in searchDropdown" :key="item" :value="item"/>
          </el-select>
        </template>
        <template #default="scope">
          {{ scope.row.desc }}
        </template>
      </el-table-column>
      <el-table-column>
        <template #header>
          <el-select v-model="tag" placeholder="搜索" filterable clearable allow-create>
            <el-option v-for="item in tagDropdown" :key="item" :value="item"/>
          </el-select>
        </template>
        <template #default="scope">
          {{ scope.row.tags }}
        </template>
      </el-table-column>
      <el-table-column sortable prop="total" label="总价" width="65">
        <template #default="scope">
          <span :class="typeof scope.row.class == 'string' ? scope.row.class: ''">{{ scope.row.total }}</span>
        </template>
      </el-table-column>
      <el-table-column prop="unit" sortable label="单价" width="80"  />
    </el-table>
  </div>
</template>

<script>
const axios = require('axios');
import ClipboardJS from 'clipboard'

export default {
  name: 'App',
  data() {
    return {
      loading: true,
      showGetJson: false,
      getAll: true,
      houseId: '%252Fl3c2113333184394102',
      housePage: 1,
      table: [],
      file: '',
      file2: '',
      files: [],
      search: '',
      searchDropdown: ['2室', "3室", "东", "南", "西", "北"],
      tag: '',
      tagDropdown: ['满两年','满五年'],
      area: 'zhishicheng',
      compareResult: [],
      copyJson: {}
    }
  },
  methods: {
    getUrlVar(url, key) {
      var result = new RegExp("[?|&]" + key + "=([^&]*)", "i").exec(url); return result && unescape(result[1]) || "";
    },
    houseIdChange(){
      this.table = [];
      this.copyJson = {};
      this.housePage = 1;
    },
    areaChange(data){
      this.getFiles(data);
    },
    dataChange(data) {
      try{
        const json = JSON.parse(data)
        if(typeof json == 'object'){
          this.table = json.data;
          console.log(this.table)
        }
      }catch(e){
        //
      }
    },
    getFiles(data) {
      const $this = this;
      axios.get('http://tao.xgzuche.com/house/json/' + data + '/')
      .then(function (response) {
        $this.files = response.data
      })
      .catch(function (error) {
        console.log(error);
      })
      .then(function () {
        $this.loading = false
      });
    },
    fileChange(data) {
      this.search = ''
      document.title = data
      const $this = this
      this.compareResult = []
      $this.loading = true
      axios.get('http://tao.xgzuche.com/house/json/'+this.area+'/?name=' + encodeURIComponent(data))
      .then(function (response) {
        $this.table = response.data.data
      })
      .catch(function (error) {
        console.log(error);
      })
      .then(function () {
        $this.loading = false
      });
    },
    compare() {
      const table = this.table;
      const tableStr = JSON.stringify(table)
      if(table.length == 0) return;
      const $this = this
      let data = this.file2;
      $this.loading = true
      axios.get('http://tao.xgzuche.com/house/json/'+this.area+'/?name=' + encodeURIComponent(data))
      .then(function (response) {
        data = response.data.data
        let compareResult = [];
        for (let i = 0; i < data.length; i++) {
          const item = data[i];
          const id = item.id;
          for (let a = 0; a < table.length; a++) {
            const item2 = table[a];
            const id2 = item2.id;
            if (id == id2) {
              let tmpItem = {
                class: '',
                id: id,
                link: item.link,
                img: item.img,
                title: item.title,
                desc: item.desc,
                tags: item.tags,
                total:  item2.total + '\n' + item.total,
                unit:  item2.unit + '\n' + item.unit,
              };
              if(item2.total != item.total){
                tmpItem.class = item.total > item2.total ? 'red' : 'green';
              }
              compareResult.push(tmpItem);
            }else if(a == 0 && tableStr.indexOf(item.id) == -1){
              compareResult.push(item);
            }
          }
        }
        $this.compareResult = compareResult;
      })
      .catch(function (error) {
        console.log(error);
      })
      .then(function () {
        $this.loading = false
      });
    },
    getJson(){
      const $this = this;
      const houseId = this.houseId;
      if(houseId == ''){
        this.$message({
          showClose: true,
          message: '请选择房屋',
          type: 'error'
        });
        return;
      }
      const housePage = this.housePage;
      $this.$message({
        showClose: true,
        duration: 600,
        offset: 100,
        message: '正在获取第'+housePage+'页数据'
      });

      $this.loading = true;
      this.compareResult = [];
      if($this.housePage == 1 || $this.getAll == false) $this.table = [];
      axios.get('http://tao.xgzuche.com/house/json/getCurl.php?url=' + encodeURIComponent('https://m.ke.com/liverpool/api/ershoufang/getList?cityId=440100&condition=' + houseId + '&curPage=' + housePage))
      .then(function (response) {
        if(response.data.status == 1){
          const data = response.data.data;
          const list = data.data.data.getErShouFangList.list
          if(list.length == 0){
            if($this.getAll && $this.housePage != 1){
              $this.$message({
                showClose: true,
                message: '获取完毕 共获取到' + $this.table.length + '个数据',
                type: 'success'
              });
            }else{
              $this.$message({
                showClose: true,
                message: '获取失败',
                type: 'error'
              });
            }
          }else{
            let arr = [];
            for (let i = 0; i < list.length; i++) {
              const obj = list[i];
              let tmp = {
                "id": obj.houseCode,
                "link": obj.jumpUrl,
                "img": obj.listPictureUrl,
                "title": obj.title,
                "desc": obj.desc,
                "total": obj.totalPrice,
                "unit": obj.unitPrice.replace('元/平','')
              };
              arr.push(tmp);
            }
            $this.table = $this.table.concat(arr);
            let now = new Date();
            $this.copyJson = {
              'data' : $this.table,
              'date' : now.getFullYear() + "-" +((now.getMonth()+1)<10?"0":"")+(now.getMonth()+1)+"-"+(now.getDate()<10?"0":"")+now.getDate()
            };
            if($this.getAll){
              $this.housePage++;
              $this.getJson();
            }
          }
        }else{
          $this.$message({
            showClose: true,
            message: '获取失败',
            type: 'error'
          });
        }
      })
      .catch(function (error) {
        console.log(error);
      })
      .then(function () {
        $this.loading = false
      });
    }
  },
  mounted() {
    const $this = this;
    this.showGetJson = this.getUrlVar(window.location.href,'debug');
    this.getFiles(this.area)
    this.clipboard = new ClipboardJS('.copy')
    this.clipboard.on('success', function() {
      $this.$message({
        showClose: true,
        message: '复制成功',
        duration: 600,
        type: 'success'
      });
    })
  },
  unmounted() {
    // 销毁Clipboard实例，避免在其它页面或组件中实例化Clipboard后造成再次监听，产生重复回调
    this.clipboard.destroy()
  },
}
</script>

<style>
body{
  margin: 0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  padding: 20px 0;
  max-width: 730px;
  margin: 0 auto;
  font-size: 14px;
}
#top{
  display: flex
}
#data{
  margin-bottom: 20px;
}
.get{
  margin-bottom: 20px;
  display: flex;
  justify-content: space-between;
}
.compare{
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}
.el-table{
  font-size: inherit;
}
.compare .el-select{
  width: 30%;
}
.el-button.is-round{
  height: 40px;
}
.el-table .cell{
  white-space: pre-line;
  line-height: 1.4;
  padding-left: 7px;
  padding-right: 7px;
}
.red{
  color: red;
}
.green{
  color: green;
}
@media print{
  #app{
    font-size: 12px;
  }
  .el-table .el-table__cell{
    padding: 5px 0;
  }
  #data,.caret-wrapper{
    display: none!important;
  }
}
</style>
