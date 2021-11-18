<template>
  <div v-loading="loading">
    <meta name="referrer" content="never">
    <!-- <el-input id="data" v-model="data" :rows="4" type="textarea" placeholder="Please input" @change="dataChange" /> -->
    <div class="compare">
      <el-select v-model="area" placeholder="区域"  @change="areaChange">
        <el-option label="知识城" value="zhishicheng"/>
        <el-option label="番禺广场" value="panyuguangchang"/>
      </el-select>
      <el-select id="select" filterable clearable v-model="file" placeholder="Select" @change="fileChange">
        <el-option v-for="file in files" :key="file" :label="file" :value="file" />
      </el-select>
      <el-select id="select" filterable clearable v-model="file2" placeholder="Select">
        <el-option v-for="file in files" :key="file" :label="file" :value="file" />
      </el-select>
      <el-button type="primary" round @click="compare">对比</el-button>
    </div>
    <el-table :data="compareResult.length==0 ?table:compareResult" stripe style="width: 100%" :default-sort="{ prop: 'total', order: 'ascending' }">
      <el-table-column prop="id" sortable label="编号" width="70" />
      <el-table-column label="图片" width="60">
        <template #default="scope">
          <a :href="scope.row.link" target="_blank"><img :src="scope.row.img" width="60"></a>
        </template>
      </el-table-column>
      <el-table-column prop="title" label="标题"/>
      <el-table-column prop="desc" label="描述" />
      <el-table-column prop="tags" label="标签" />
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

export default {
  name: 'App',
  data() {
    return {
      loading: true,
      table: [],
      file: '',
      file2: '',
      files: [],
      area: 'zhishicheng',
      compareResult: []
    }
  },
  methods: {
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
  },
  mounted() {
    this.getFiles(this.area)
  }
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
.compare{
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}
.el-table{
  font-size: inherit;
}
.el-select{
  width: 40%;
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
