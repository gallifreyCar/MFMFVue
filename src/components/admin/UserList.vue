<template>
  <div>
    <h2>用户列表</h2>
<!--  面包屑导航-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
<!--      用户列表主体-->
      <el-card>
        <el-row :gutter="25"> <!--间隙-->
<!--        搜索区域-->
<!--        搜索添加-->
          <el-col :span="10" >  <!--列宽-->
            <el-input placeholder="请输入内容"  v-model="queryInfo.query" clearable @click="getUserList">
              <el-button slot="append" icon="el-icon-search" @click="getUserList">

              </el-button>
            </el-input>
          </el-col>
          <!-- 按钮-->
          <el-col :span="4">
            <el-button type="primary" @click="addDialogVisible =true">添加用户</el-button>
          </el-col>
        </el-row>
        <!--用户列表   border边框   stripe隔行变色-->
        <el-table :data="userList" border stripe>
          <el-table-column type="index"></el-table-column> <!--索引列-->
          <el-table-column label="用户名" prop="username"></el-table-column>
          <el-table-column label="昵称" prop="name"></el-table-column>
          <el-table-column label="邮箱" prop="email"></el-table-column>
          <el-table-column label="所持花表" prop="flowerId"></el-table-column>

          <el-table-column label="状态" prop="state">
            <!--作用域插槽-->
            <template slot-scope="scope">
<!--              {{scope.row}}每一行封存的数据-->
              {{scope.row.state}}
              <el-switch v-model="scope.row.state" @change="userStateChange(scope.row)"></el-switch>
            </template>
          </el-table-column>

          <el-table-column label="操作" >
            <template slot-scope="scope">

              <!-- 修改 -->
              <el-tooltip effect="dark" content="修改信息" placement="top-start" :enterable="false" >
              <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog2(scope.row)"></el-button>
              </el-tooltip>

              <!-- 删除 -->
              <el-tooltip effect="dark" content="删除用户" placement="top-start" :enterable="false">
              <el-button type="danger" icon="el-icon-delete" size="mini" @click= "deleteUser(scope.row.id)"></el-button>
              </el-tooltip>

              <!-- 权限 -->
              <el-tooltip effect="dark" content="分配权限" placement="top-start" :enterable="false"><!--文字提示 enterable 隐藏-->
                <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>

<!--        分页组件 size-change每页最大变化数  current-change当前变化数 layout功能组件-->
        <div>
          <el-pagination
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="queryInfo.pageNum"
              :page-sizes="[1, 2, 5,10 ,20,100]"
              :page-size="queryInfo.pageSize"
              layout="total, sizes, prev, pager, next, jumper"
              :total="total">
          </el-pagination>
        </div>
      </el-card>

    <!--新增用户表对话框-->
    <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClosed">
      <el-form  ref="addFormRef" :model="addForm" :rules="addFormRules" label-width="90px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password" >
          <el-input v-model="addForm.password" show-password></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="昵称" prop="name">
          <el-input v-model="addForm.name"></el-input>
        </el-form-item>

       <el-form-item label="初始花表" prop="flowerId">
          <el-select v-model="addForm.flowerId" placeholder="请选择初始花表">
            <el-option label="1-牵牛花" value=1></el-option>
            <el-option label="2-玫瑰  " value=2></el-option>
            <el-option label="3-荷兰菜花" value=3></el-option>
           <el-option label="4-豆腐花" value=4></el-option>
          </el-select>
       </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!--修改用户表对话框-->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
      <el-form  ref="editFormRef" :model="editForm" :rules="editFormRules" label-width="90px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="editForm.username" disabled></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password" >
          <el-input v-model="editForm.password" show-password></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="editForm.email"></el-input>
        </el-form-item>
        <el-form-item label="昵称" prop="name">
          <el-input v-model="editForm.name"></el-input>
        </el-form-item>

        <el-form-item label="初始花表" prop="flowerId" >
          <el-select v-model="editForm.flowerId" placeholder="请选择初始花表"disabled>
            <el-option label="1-牵牛花" value=1></el-option>
            <el-option label="2-玫瑰  " value=2></el-option>
            <el-option label="3-荷兰菜花" value=3></el-option>
            <el-option label="4-豆腐花" value=4></el-option>
          </el-select>
        </el-form-item>
      </el-form>

      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUser">确 定</el-button>
      </span>
    </el-dialog>

  </div>
</template>

<script>
export default {
  //类似构造器
  created() {
    this.getUserList()
  },
  //数据返回 创建属性
  data(){
    // //自定义的邮箱和手机验证规则
    // let checkEmail = (rule, value,callback) =>{//验证邮箱
    //   const regEmail = /^([a-zA-Z]|[0-9])(\w|\-)+@[a-zA-Z0-9]+\.([a-zA-Z]{2,4})$/;
    //   if(this.addForm.email != '' && !regEmail.test(this.addForm.email)) {
    //     callback(new Error('请输入有效的邮箱'));
    //   }
      //自定义的邮箱和手机验证规则
      let checkEmail = (rule, value, callback) => {
        rule = /^\w{3,12}@\w{1,5}\.[a-z]{2,3}$/;
        if (!value) {
          callback(new Error('邮箱不能为空'));
        } else {
          if (!rule.test(value)) {
            callback(new Error('输入的邮箱格式有误！'));
          } else {
            callback();
          }
        }
      }



    return{

      //查询信息实体
      queryInfo:{
        query:"",//查询信息
        pageNum:1,//当前页
        pageSize:5,//每页最大数
      },
      userList:[],//用户列表
      total:0,//总记录数
      addDialogVisible:false,//对话框状态

      //添加表单信息
      addForm:{
        username:'',
        password:'',
        email:'',
        flowerId:'',
        name:'',

      },

      //修改用户信息
      editForm:{},
      //显示/隐藏修改用户栏
      editDialogVisible:false,

      //添加用户表单验证
      addFormRules: {
        username: [
          {required: true, message: '请输入昵称', trigger: 'blur'},
          {min: 5, max: 12, message: '长度在 5 到 12 个字符', trigger: 'blur'}
        ],
        name: [
          {required: true, message: '请输入昵称', trigger: 'blur'},
          {min: 2, max: 12, message: '长度在 2 到 12 个字符', trigger: 'blur'}
        ],
        password: [
          {required: true, message: '请输入密码', trigger: 'blur'},
          {min: 6, max: 10, message: '长度在 6 到 10 个字符', trigger: 'blur'}
        ],
         email: [
           {required: true, message: '请输入邮箱', trigger: 'blur'},
            // {min: 6, max: 14, message: '请输入正确的邮箱', trigger: 'blur'}
            {validator: checkEmail, trigger: "blur" }
         ],
         flowerId: [
           {required: true, message: '请选择初始花表', trigger: 'change'}
         ],
      },

      //修改用户表单验证
      editFormRules: {
        name: [
          {required: true, message: '请输入昵称', trigger: 'blur'},
          {min: 2, max: 12, message: '长度在 2 到 12 个字符', trigger: 'blur'}
        ],
        password: [
          {required: true, message: '请输入密码', trigger: 'blur'},
          {min: 6, max: 10, message: '长度在 6 到 10 个字符', trigger: 'blur'}
        ],
        email: [
          {required: true, message: '请输入邮箱', trigger: 'blur'},
          {min: 6, max: 14, message: '请输入正确的邮箱', trigger: 'blur'}
        ],
        flowerId: [
          {required: false, message: '请选择初始花表', trigger: 'change'}
        ],
      },
    };
  },

  //方法
  methods:{



    // 获取所有用户
    async getUserList(){
      const{data:res} =await this.$http.get("allUser",{params:this.queryInfo})
      this.userList=res.data;//用户列表数据封装
      this.total=res.numbers;//总用户数封装
    },


    //处理分页页面变化 最大数
    handleSizeChange(newSize){
      this.queryInfo.pageSize=newSize;
      this.getUserList();
    },

    //pageNum的触发动作
    handleCurrentChange(newPage){
      this.queryInfo.pageNum=newPage;
      this.getUserList();
    },



    //修改用户状态
    async userStateChange(userInfo){
       const{data:res} =await this.$http.post(`userState?id=${userInfo.id}&state=${userInfo.state}`);
      // var formData = new FormData();
      // formData.append('id', userInfo.id);
      // formData.append('state', userInfo.state);
      // const{data:res}= await this.$http.put('/userState', formData);
       if(res!="success"){
         userInfo.id = !userInfo.id;
        return this.$message.error("操作失败");
       }
       this.$message.success("操作成功")
    },

    // 监听用户的操作
    addDialogClosed(){
        this.$refs.addFormRef.resetFields();
    },

    editDialogClosed(){
      this.$refs.editFormRef.resetFields();
    },


    //添加用户
    addUser(){
      this.$refs.addFormRef.validate(async valid=>{
            console.log(valid);
            // 验证后台，不成功返回
             if(!valid) return;
             const{data:res}=await this.$http.post("addUser",this.addForm);
             if(res!="success"){
               return this.$message.error("操作失败");
             }
              this.$message.success("操作成功");
              this.addDialogVisible = false;
              await this.getUserList();
          });
    },

    //删除用户
    async deleteUser(id){
     const confirmResult =await this.$confirm("此操作将永久删除用户，是否继续？",'警告',{
       confirmButtonText:"确定",
       cancelButtonClass:"取消",
       type:'warning'
     }).catch(err=>err)
      if(confirmResult!='confirm'){
        return this.$message.info("已取消删除")
      }
      const{data:res}=await this.$http.post("deleteUser?id="+id);
      console.log(res);
      if(res!="success"){
        return this.$message.error("操作失败");
      }
      this.$message.success("操作成功");
      await this.getUserList();
    },

    // //显示对话框
    // async showEditDialog(id){
    //    const {data:res}=await this.$http.get("getUpdateUser?id="+id);
    //    this.editForm=res;//查询出用户信息反填到编辑表单里面
    //     this.editDialogVisible=true;//开启编辑对话框
    // },

     //显示对话框2
     async showEditDialog2(Info){
       this.editForm=Info;//查询出用户信息反填到编辑表单里面
       this.editDialogVisible=true;//开启编辑对话框
     },

   //修改用户
  editUser(){
    this.$refs.editFormRef.validate(async valid=>{
      console.log(valid);
      // 验证后台，不成功返回
      if(!valid) return;
      const{data:res}=await this.$http.post("editUser",this.editForm);
      if(res!="success"){
        return this.$message.error("操作失败");
      }
      this.$message.success("操作成功");
      this.editDialogVisible = false;
      await this.getUserList();

    })
  }
}

};
</script>

<style lang='less' scoped>
.el-breadcrumb{
  margin-bottom: 15px;
  font-size:17px;
}
.el-select .el-input {
  width: 130px;
}

.el-row{

}
</style>