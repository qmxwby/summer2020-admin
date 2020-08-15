<template>
    <div>
        <div class="crumbs">
            <el-breadcrumb separator="/">
                <el-breadcrumb-item>
                    <i class="el-icon-lx-cascades"></i> 教师管理
                </el-breadcrumb-item>
            </el-breadcrumb>
        </div>
        <div class="container">
            <div class="handle-box">
                <el-button
                    type="primary"
                    icon="el-icon-delete"
                    class="handle-del mr10"
                    @click="delAllSelection"
                >批量删除</el-button>
				<el-button
				    type="success"
				    icon="el-icon-edit"
				    class="handle-del mr10"
				    @click="addTeacher"
				>添加教师</el-button>
                <el-input v-model="query.name" placeholder="输入教师姓名" class="handle-input mr10"></el-input>
                <el-button type="primary" icon="el-icon-search" @click="handleSearch">搜索</el-button>
            </div>
            <el-table
                :data="tableData"
                border
                class="table"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange"
            >
                <el-table-column type="selection" width="55" align="center"></el-table-column>
                <el-table-column prop="tid" label="教师编号" width="200" align="center"></el-table-column>
                <el-table-column prop="tname" label="姓名" width="150"></el-table-column>
                <el-table-column prop="tsex" label="性别" width="80"></el-table-column>
                <el-table-column prop="deptName" label="学院" width="200"></el-table-column>
                <el-table-column prop="tprof" label="职称"></el-table-column>
                <el-table-column prop="tinyear" label="入职年份"></el-table-column>
                <el-table-column label="操作" width="180">
                  <template slot-scope="scope">
                    <el-tooltip class="item" effect="light" open-delay="500" content="更新" placement="bottom" :enterable="false">
                    <el-button type="primary" size="mini" icon="el-icon-edit"
                    circle
                    @click="updateOneTeacher(scope.row.tid, scope.row)"
					style="margin-left: 30px;"></el-button>
                    </el-tooltip>
                    <el-tooltip class="item" effect="light" open-delay="500" content="删除" placement="bottom" :enterable="false">
                    <el-button
                      type="danger"
                      icon="el-icon-delete"
                      size="mini"
                      circle
                      @click="removeOneTeacher(scope.row.id)"
					 style="margin-left: 30px;"
                    ></el-button>
                    </el-tooltip>
                  </template>
                </el-table-column>
            </el-table>
            <div class="pagination">
				<el-pagination
				  @size-change="handleSizeChange"
				  @current-change="handleCurrentChange"
				  :current-page="query.pageIndex"
				  :page-sizes="[5, 10, 20, 40]"
				  :page-size="query.pageSize"
				  layout="total, sizes, prev, pager, next, jumper"
				  :total="pageTotal">
				</el-pagination>
            </div>
        </div>

        <!-- 编辑弹出框 -->
        <el-dialog title="编辑教师信息" :visible.sync="editVisible" width="30%">
            <el-form ref="form" :model="form" label-width="70px">
                <el-form-item label="教师编号">
                    <el-input v-model="form.tid" disabled></el-input>
                </el-form-item>
				<el-form-item label="入职年份">
				    <el-input v-model="form.tinyear" disabled></el-input>
				</el-form-item>
                <el-form-item label="姓名">
                    <el-input v-model="form.tname"></el-input>
                </el-form-item>
				<el-form-item label="性别">
				     <el-radio v-model="form.tsex" label="男">男</el-radio>
					 <el-radio v-model="form.tsex" label="女">女</el-radio>
				</el-form-item>
				<el-form-item label="学院">
					<el-select v-model="form.deptName" placeholder="请选择活动区域">
					  <el-option v-for="item in options"
					  :value="item.value"
					  :label="item.label"
					  ></el-option>
					</el-select>
				</el-form-item>
				<el-form-item label="职称">
				    <el-input v-model="form.tprof"></el-input>
				</el-form-item>	
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editVisible = false">取 消</el-button>
                <el-button type="primary" @click="saveEdit">确 定</el-button>
            </span>
        </el-dialog>
    </div>
</template>

<script>
export default {
    name: 'basetable',
    data() {
        return {
            query: {
                pageIndex: 1,
                pageSize: 5
            },
			options: [{
			          value: '11',
			          label: '计算机与信息学院'
			        }, {
			          value: '12',
			          label: '土木与建筑学院'
			        }],
            tableData: [],
            multipleSelection: [],
            delList: [],
            editVisible: false,
            pageTotal: 0,
            form: {},
            idx: -1,
            id: -1
        };
    },
    created() {
        this.getData();
    },
    methods: {
        // 获取 后台教师数据
        getData() {
			this.$http.get('teacher/getAll/'+this.query.pageIndex+'/'+this.query.pageSize)
			.then(resp => {
				let res = resp.data;
				console.log(res);
				if(res.code == 200) {
					this.tableData = res.data.records;
					this.pageTotal = res.data.total;
				} else {
					this.$message.error(res.msg);
				}
			});
        },
        // 触发搜索按钮
        handleSearch() {
            this.$set(this.query, 'pageIndex', 1);
            this.getData();
        },
        // 删除操作
        handleDelete(index, row) {
            // 二次确认删除
            this.$confirm('确定要删除吗？', '提示', {
                type: 'warning'
            })
                .then(() => {
                    this.$message.success('删除成功');
                    this.tableData.splice(index, 1);
                })
                .catch(() => {});
        },
        // 多选操作
        handleSelectionChange(val) {
            this.multipleSelection = val;
        },
        delAllSelection() {
        },
		//改变页码
		handleCurrentChange(val) {
			this.query.pageIndex = val;
			this.getData();
		},
		//改变当前显示数量
		handleSizeChange(val) {
			this.query.pageSize = val;
			this.getData();
		},
		//编辑事件
		updateOneTeacher(tid, data) {
			this.editVisible = true;
			this.form = data;
			this.$http.get('teacher/getById/'+tid)
			.then(resp => {
				let res = resp.data;
				if(res.code == 200){
					this.form.tpassword = res.data.tpassword;
					console.log(this.form);
				} else {
					this.$message.error(res.msg);
				}
			});		
		},
		//保存教师编辑对话框
		saveEdit() {
			this.editVisible = false;
			this.$http.post('teacher/update', this.form)
			.then(resp => {
				let res = resp.data;
				if(res.code == 200){
					this.$message.success(res.msg);
					this.getData();
				} else {
					this.$message.error('请求失败！');
				}
			});		
		}
    }
};
</script>

<style scoped>
.handle-box {
    margin-bottom: 20px;
}

.handle-select {
    width: 120px;
}

.handle-input {
    width: 300px;
    display: inline-block;
}
.table {
    width: 100%;
    font-size: 14px;
}
.red {
    color: #ff0000;
}
.mr10 {
    margin-right: 10px;
}
.table-td-thumb {
    display: block;
    margin: auto;
    width: 40px;
    height: 40px;
}
</style>
