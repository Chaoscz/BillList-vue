<template>
	<div>
		<el-row>
		  <el-col :span="12">
			<el-input v-model="input" placeholder="api?method=xx"></el-input>
		  </el-col>
		  <el-col :span="12">
		  	 <el-button type="primary" icon="el-icon-search" @click="search()">搜索</el-button>
		  </el-col>
		</el-row>

		<el-row style="margin-top: 12px;">
			<el-col :span="12">
			<el-card class="box-card">
			  <div slot="header" class="clearfix">
			    <span>搜索结果：</span>
			  </div>
			  <div v-for="o in data"  class="text item">
			    {{ o }}
			  </div>
			</el-card>
			</el-col>
		</el-row>
	</div>		
</template>
<script>
export default {
  data() {
    return {
      input: '',
      data:'',
    }
  },
  methods:{
  	search:function(){
  	  console.log("开始搜索api...")
  	  var url = this.input
  	  console.log(url)
	  this.$axios.get(url).then((res) => {
	  	console.log(res.data)
      this.data = res.data
      })
  	}
  }
}
</script>
<style>
	
</style>