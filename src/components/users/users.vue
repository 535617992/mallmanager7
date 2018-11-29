<template>
  <el-card class="box-card">
   <!--1. 面包屑 -->
    <el-breadcrumb separator="/">
        <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item><a href="/">用户管理</a></el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
   <!--2.搜索框 -->
   <el-row class="search">
       <el-col :span='12' >
            <el-input clearable v-model="query" placeholder="请输入内容"  class="input-with-select  aa" >
                <el-button @click="searchUser()" slot="append" icon="el-icon-search"></el-button>
            </el-input> 
       </el-col>
           <el-button @click="showAddUserDia()" type="success">添加按钮</el-button>   
   </el-row>
   <!--3.用户列表 -->
   <el-table
      :data="userlist"
      style="width: 100%">
      <el-table-column
       type="index"
        label="#"
        width="80"
       >
      </el-table-column>
      <el-table-column
        prop="username"
        label="姓名"
        width="100">
      </el-table-column>
      <el-table-column
        prop="email"
         width="200px"
        label="邮箱">
      </el-table-column>
      <el-table-column
        prop="mobile"
        label="电话">
      </el-table-column>
      <el-table-column label="创建日期">
        <template slot-scope="scope">
            <!-- {{create_time | fmtdate}} -->
            <!-- userlist中的每个对象中的create_time -->
            {{scope.row.create_time |fmtdate}}
        </template>  
      </el-table-column>
      <el-table-column
        prop="mg_state"
        label="用户状态">
        <template slot-scope="scope">
            <el-switch
            @change="changeUserMg_state(scope.row)"
            v-model="scope.row.mg_state"
            active-color="#13ce66"
            inactive-color="#ff4949">
        b    </el-switch>
        </template> 
               
      </el-table-column>

      <el-table-column  label="操作" width="180px">
          <template slot-scope="scope">
            <el-row>
            <el-button size="mini"  plain type="primary" icon="el-icon-edit"
             @click="showEditUserDia(scope.row)"
             circle>
             </el-button>
            <el-button size="mini"  plain type="danger" icon="el-icon-delete"
            @click="showMegBoxDele(scope.row.id)"
             circle>
            </el-button>
              <el-button size="mini"  plain type="success" icon="el-icon-check" 
            @click="showSetRoleDia(scope.row)"  
              circle></el-button>
            </el-row>
          </template>
      </el-table-column> 
    </el-table>
   <!--4.分页-->
   <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="pagenum"
      :page-sizes="[2, 4, 6, 8]"
      :page-size="2"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    <!-- 5.添加用户对话框 -->
     <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdd">
        <el-form :model="form">
            <el-form-item label="用户名称" label-width="100px">
                <el-input v-model="form.username" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="密码" label-width="100px">
                <el-input v-model="form.password" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" label-width="100px">
                <el-input v-model="form.email" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="电话" label-width="100px">
                <el-input v-model="form.mobile" autocomplete="off"></el-input>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleAdd = false">取 消</el-button>
            <el-button type="primary"
            @click="addUser()">确 定</el-button>
        </div>
        </el-dialog>
         <!-- 6.编辑用户对话框 -->
     <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdit">
        <el-form :model="form">
            <el-form-item label="用户名称" label-width="100px">
                <el-input  disabled v-model="form.username" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="邮箱" label-width="100px">
                <el-input v-model="form.email" autocomplete="off"></el-input>
            </el-form-item>
            <el-form-item label="电话" label-width="100px">
                <el-input v-model="form.mobile" autocomplete="off"></el-input>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleEdit= false">取 消</el-button>
            <el-button type="primary"  @click="editUser()"
            >确 定</el-button>
        </div>
        </el-dialog>
        <!-- 7分配角色对话框 -->
        <el-dialog title="分配角色" :visible.sync="dialogFormVisibleRole">
        <el-form :model="form">
            <el-form-item label="用户名" label-width="100px">
                {{currUsername}}
            </el-form-item>
            <el-form-item label="角色" label-width="100px">
            <el-select v-model="currRoleId" >
                <el-option label="请选择" :value="-1"></el-option>
                 <el-option v-for="(item,i) in roles " :key="i" :label="item.roleName" :value="item.id"></el-option>           
            </el-select>
            </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
            <el-button @click="dialogFormVisibleRole= false">取 消</el-button>
            <el-button type="primary" @click="setRole()">确 定</el-button>
        </div>
        </el-dialog>

  </el-card>
</template>

<script>
export default {
    data(){
         return{
        query:'',
        userlist:[{
            id:'',
            username:'',
            email:'',
            mobile:'',
            create_time:'',
            mg_state:'',
            role_name:''

        }],
        pagenum:1,
        pagesize:2,
        total:-1,
        //添加对话框
        dialogFormVisibleAdd:false,
        //编辑对话框
       dialogFormVisibleEdit:false,
        //分配角色对话框
        dialogFormVisibleRole:false,

        //用户表单数据
        form:{
            username:'',
            password:'',
            email:'',
            mobile:''
        },
        //分配角色
        //当前用户的用户名
        currUsername:'',
        //当前角色名称
        currRoleId:'-1',
        //保存返回的角色数组，数组中的每一项都是开一个对象
        roles:[],
        //当前用户的ID
        currUserId:-1
    }  
    },

    //方法可以在mounted中调用，也可以在created中调用
    created(){
         this.getUserList()
     },

    methods:{
        //发送请求修改用户角色
      async  setRole(){
          const res=await this.$http.put(`users/${this.currUserId}/role`,{ rid:this.currRoleId})
        console.log(res)
         this.dialogFormVisibleRole=false
        },

        //分配设置用户角色
       async showSetRoleDia(user){
           //给当前的用户id赋值
           this.currUserId=user.id


            this.currUsername=user.username
            this.dialogFormVisibleRole=true
            //发送请求获取所有角色名称
            const res1=await  this.$http.get('roles')
           //   console.log(res1)
           this.roles=res1.data.data
            //显示当前用户的角色  ID rid
            const res2=await this.$http.get(`users/${user.id}`)
            console.log(res2)
            this. currRoleId=res2.data.data.rid
            


        },
        //修改用户状态
       async changeUserMg_state(user){
       const res=await this.$http.put(`users/${user.id}/state/${user.mg_state}`)
        console.log(res)
        },

        //编辑  发送请求
       async editUser(){
           const res= await this.$http.put(`users/${this.form.id}`,this.form)
        //    console.log(res)
           const {meta:{msg,status}}=res.data
           if(status===200){
               this.$message.success(msg)
               this.dialogFormVisibleEdit=false
               this.getUserList()

           }
        },
        //编辑  打开编辑对话框、
        showEditUserDia(user){
            this.form=user
            
            this.dialogFormVisibleEdit=true
        },
     //删除  打开消息提示框
       showMegBoxDele(usersId){
        this.$confirm('是否删除该用户?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(async() => {
            //点击确定发送请求 更新视图
          const res=await this.$http.delete(`users/${usersId}`)

          this.getUserList()
          this.$message({
            type: 'success',
            message: res.data.meta.msg
          });
        }).catch(() => {
          this.$message({
            type: 'info',
            message: res.data.meta.msg
          });          
        });
      },




       
        // 添加用户 - 发送请求
    async addUser() {
      // 2. 关闭对话框
      this.dialogFormVisibleAdd = false

      // 1. 发送请求
      const res = await this.$http.post('users', this.form)
    //   console.log(res)
      const {
        meta: { status, msg },
        data
      } = res.data

      if (status === 201) {
        // 3. 提示成功
        this.$message.success(msg)
        // 4. 更新视图
        this.getUserList()
        // 5. 清空input
        this.form = {}

             } else {
        this.$message.warning(msg)
      }
    },
        // 添加用户 - 显示对话框
        showAddUserDia() {
        this.dialogFormVisibleAdd = true
        },
        
        // ${this.query}es6模板字符串
       async getUserList(){
        // 需要授权的 API ，必须在请求头中使用 Authorization 字段提供 token 令牌
          // 设置请求头 进行授权
      let AUTH_TOKEN = localStorage.getItem('token')
      this.$http.defaults.headers.common['Authorization'] = AUTH_TOKEN
         
         
      const res= await this.$http.get(`users?query=${this.query}&pagenum=${this.pagenum}&pagesize=${this.pagesize}`)
    //    console.log(res)
       const {meta:{msg,status},data:{total,users}}=res.data
        if(status===200){
            this.total=total
            this.userlist=users
            this.$message.success(msg)
        }else{
            this.$message.warning(msg)
        }
      },
      //分页中相关方法
      handleSizeChange(val) {
        console.log(`每页 ${val} 条`);
        this.pagesize=val
        this.pagenum=1
        this.getUserList()
      },
      handleCurrentChange(val) {
        console.log(`当前页: ${val}`);
        this.pagenum=val;
        this.getUserList()
      },
       //搜索用户
        searchUser(){
            this.pagenum=1
            this.getUserList()

        }
    
    }
}
</script>

<style>
.search{
    margin-top: 30px
}
.aa{
    width: 500px;
}
</style>
