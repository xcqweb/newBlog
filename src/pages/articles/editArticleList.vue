<template>
		<div class="articleList" v-loading='loading'>
			<!--搜索-->
			<div class="search">
				<span style="font-size: 14px;margin-right: 10px;">类别 :</span>
				<el-select v-model="v1" filterable placeholder="请选择" size='mini'>
				    <el-option
				      v-for="item in options"
				      :key="item.value"
				      :label="item.label"
				      :value="item.value">
				    </el-option>
				</el-select>
				
				<span style="font-size: 14px;margin: 0 10px 0 10px;">关键词搜索 :</span>
				<el-input v-model="v2" style='width: 200px;margin-right: 10px;' size='mini' placeholder="请输入关键词"></el-input>
				
				<el-button
		          size="mini"
		          type='primary'
		          icon="el-icon-search"
		          @click="search">
		                 搜索
		          </el-button>
			</div>
			
			<p v-show="ids"  style="width: 96%;margin: auto;">
				 <el-button
		          size="mini"
		          type="danger"
		          @click="delMore">删除多项</el-button>
			</p>
			
			<!--表格-->
	  		<el-table
		    ref="multipleTable"
		    :data="tableData"
		    tooltip-effect="dark"
		    style="width: 96%;margin: 36px auto;text-align: center;min-width: 1000px;"
		    border
		    stripe
		    show-overflow-tooltip
		    @selection-change="handleSelectionChange"
		    >
		    <el-table-column
		      type="selection"
		      width="55">
		    </el-table-column>
		    <el-table-column
		      type="index"
		      width="55"
		      >
		    </el-table-column>
		    <el-table-column
		      label="标题"
		      prop="title"
		      >
		    </el-table-column>
		    <el-table-column
		      prop="author"
		      label="作者"
		      width="120">
		    </el-table-column>
		    <el-table-column
		      prop="classify"
		      label="类别"
		      width='150'
		      >
		    </el-table-column>
		    
		    <el-table-column
		      prop="ctime"
		      label="发布时间"
		      width='150'
		      >
		    </el-table-column>
		    
		    <el-table-column
		      prop="discuss"
		      label="评论"
		      width='55'
		      >
		    </el-table-column>
		    
		    <el-table-column
		      prop="help"
		      label="赞赏"
		      width='55'
		      >
		    </el-table-column>
		    
		    <el-table-column label="操作" width='260'>
		      <template slot-scope="scope">
		        <el-button
		          size="mini"
		          type='primary'
		          @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
		        <el-button
		          v-show = '!ids'
		          size="mini"
		          type="danger"
		          @click="handleDelete(scope.$index, scope.row)">删除</el-button>
		          
		          <el-button
		          size="mini"
		          type='primary'
		          @click="public(scope.$index, scope.row)">发布</el-button>
		      </template>
		    </el-table-column>
		</el-table>
		
		<!--分页-->
		<el-pagination
		  style='text-align: center;'
	      @current-change="handleCurrentChange"
	      :current-page="currentPage"
	      layout="total, prev, pager, next, jumper"
	      :total="total">
	    </el-pagination>
		
	</div>
</template>

<script>
	export default{
		name:'articleList',
		data(){
			return{
				tableData:[],
				ids:'',
				currentPage:1,
				total:0,
				pageSzie:0,
				loading:true,
				v1:'全部',
				v2:'',
				options: [{
		          value: '全部',
		          label: '全部'
		        },{
		          value: 'javascript',
		          label: 'javascript'
		        }, {
		          value: 'html',
		          label: 'html'
		        }, {
		          value: 'css',
		          label: 'css'
		        }, {
		          value: '框架',
		          label: '框架'
		        }, {
		          value: 'nodejs',
		          label: 'nodejs'
		        }, {
		          value: '爱生活',
		          label: '爱生活'
		        },{
		          value: '其他',
		          label: '其他'
		        }],
			}
		},
		created(){
			this.search()
		},
		activated(){
			this.search()
		},
		methods:{
			getData(params){
				this.loading = true
				this.$axios.get(this.API_URL+'/admin/temlist.html',{params:params}).then( (res) => {
					if(res.status==200){
						this.loading = false
						let len = res.data.length-1
						this.tableData = res.data
						this.total = res.data[len]
						if(this.tableData.length){
							this.tableData.pop()
						}
						
					}
				}).catch( () => {
					this.loading = false
					this.$message({
			          message: '网络出错啦!',
			          type: 'error'
			        })
				})
			},
			search(){
				let params = {
					pagesize:10,
					page:1,
					keyword:this.v2,
					classify:this.v1,
					type:1
				}
				this.getData(params)
			},
			
			handleEdit(index,item){
				this.$router.push({path:'/edit_article',query:{id:item._id,tem:true}})
			},
			handleSelectionChange(val){
				let arr = []
				for(let v of val){
					arr.push(v._id)
				}
				this.ids = arr.join(',')
			},
			delMore(){
				let params = {
					ids:this.ids
				}
				this.del(params)
			},
			handleDelete(index,item){
				let params = {
					ids:item._id
				}
				this.del(params)
			},
			del(params){
				this.$confirm('此操作将永久删除该文章, 是否继续?', '提示', {
		          confirmButtonText: '确定',
		          cancelButtonText: '取消',
		          type: 'warning'
		        }).then( () => {
		        	this.$axios.post(this.API_URL+'/admin/temdelete.html',params).then( (res) => {
						if(res.status==200){
							this.$notify({
					          message: '删除成功!',
					          type: 'success'
					        })
							this.search()
						}
					}).catch( () => {
						this.loading = false
						this.$message({
				          message: '出错啦!',
				          type: 'error'
				        })
					})
		        }).catch( () => {
		        	this.$message({
			            type: 'info',
			            message: '已取消删除'
			          });  
		        })
			},
			public : _.debounce(function(index,item){
				this.loading = true
				let params = {
					id:item._id,
					type:1
				}
				this.$axios.get(this.API_URL+'/admin/notifytemedit.html',{params:params}).then( (res) => {
					if(res.status==200){
						this.saveTem(res.data)
					}
				})
			},1000),
			
			saveTem(data){
				let params = {
					title : data.title,
					imgUrl : data.imgUrl,
					author : data.author,
					content : data.content,
					classify : data.classify,
					type:1
				}
				this.$axios.post(this.API_URL+'/admin/save_add.html',params).then( (res) => {
					if(res.data.status==200){
						this.loading = false
						this.$notify({
				          message: '恭喜你，文章发布成功!',
				          type: 'success'
				        })
						
					    this.$router.push('/article_list')
					}
				}).catch( () => {
					this.loading = false
					this.$message({
			          message: '出错啦!',
			          type: 'error'
			        })
				})
			},
			handleCurrentChange(val){
				this.loading = true
				let params = {
					pagesize:10,
					page:val,
					keyword:this.v2,
					classify:this.v1,
					type:1
				}
				this.getData(params)
			}
		}
	}
</script>

<style scoped="scoped" lang="less">
	.articleList{
		box-sizing: border-box;
		margin: 10px;
		height: 78vh;
		overflow: auto;
		.search{
			width: 96%;
			margin: 20px auto;
		}
	}
</style>