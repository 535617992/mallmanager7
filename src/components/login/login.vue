<template>
    <div class="login-wrap">
   
        <el-form 
        class="login-form"
        label-position="top" 
        label-width="80px" 
        :model="formdata"> 
            <h2>用户登录</h2>    
            <el-form-item label="用户名">
                <el-input v-model="formdata.username"></el-input>
            </el-form-item>
            <el-form-item label="密码">
                <el-input v-model="formdata.password"></el-input>
            </el-form-item>   
            <el-button class="login-btn" type="success" @click.prevent="handlelogin()">登录</el-button>
        </el-form>
       
    </div>
</template>

<script>
export default {
   data(){
       return{
           formdata:{
               username:'',
               password:''
           }
       }
   },

   methods:{
       async handlelogin(){
           const res=await this.$http.post('login',this.formdata)
           .then((res)=>{

               console.log(res)
               const {meta:{msg,status},data}=res.data

               if(status===200){
                   this.$router.push({name:'home'})
                   //提示登录成功
                   this.$message.success(msg)
               }else{
                   this.$message.warning(msg);
                  
               }
           })
       }


   }
};
</script>

<style>

.login-wrap{
    height: 100%;
    background-color: #324152;
    display: flex;
    justify-content: center;
    align-items: center
}
.login-wrap .login-form{
width: 400px;
border-radius: 15px;
background-color: #fff;
padding: 30px;
}
.login-wrap .login-btn{
    width: 100%;
}
</style>
