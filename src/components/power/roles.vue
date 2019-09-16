<template>

  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色按钮区域-->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible=true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom',i1==0?'bdtop' : '','vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeRightByID(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级权限和三级权限 -->
              <el-col :span="19">
                <!-- 通过for循环 嵌套渲染二级权限 -->
                <el-row
                  :class="[i2==0?'bdtop':'','vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag type="success" closable @close="removeRightByID(scope.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag  
                      type="warning"
                       :class="[i3==0?'bdtop':'']"
                      v-for="(item3,i3) in item2.children"
                      :key="item3.id" closable @close="removeRightByID(scope.row,item3.id)"
                    >{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <pre>
                {{scope.row }}
            </pre>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300">
          <template slot-scope="scope">
            <el-button type="primary" @click="showEditDialog(scope.row.id)" size="mini" icon="el-icon-search">编辑</el-button>
            <el-button type="danger" size="mini" icon="el-icon-delete" @click="removeUserById(scope.row.id)">删除</el-button>
            <el-button type="warning" size="mini" icon="el-icon-setting" @click="showSetRightDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
      <!-- 分配权限的对话框 -->
<el-dialog
  title="分配权限"
  :visible.sync="setRightDialogVisible"
  width="50%"
  @close="setRightDialogClosed">
  <!-- 树形控件 -->
  <el-tree :data="rightslist" node-key="id" default-expand-all :default-checked-keys="defKeys" :props="treeProps" ref="treeRef" show-checkbox></el-tree>

  <span slot="footer" class="dialog-footer">
    <el-button @click="setRightDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="allotRights">确 定</el-button>
  </span>
</el-dialog>
    <!-- 添加用户功能 -->
    <el-dialog title="添加角色" :visible.sync="addDialogVisible" @close="addDialogClosed" width="50%">
        <!-- 内容主题区 -->
    <el-form :model="addFrom" :rules="addFromRules" ref="addFormRef" label-width="80px">
  <el-form-item label="角色名称" prop="roleName">
    <el-input v-model="addFrom.roleName"></el-input>
  </el-form-item>
  <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="addFrom.roleDesc"></el-input>
  </el-form-item>
    </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 编辑用户的对话框 -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="50%" @close="editDialogClosed">
<el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="80px">
  <el-form-item label="角色名称" prop="roleName">
    <el-input v-model="editForm.roleName"></el-input>
  </el-form-item>
  <el-form-item label="角色描述" prop="roleDesc">
    <el-input v-model="editForm.roleDesc"></el-input>
  </el-form-item>

</el-form>
  <!-- 底部 -->
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editUserInfo">确 定</el-button>
  </span>
</el-dialog>

  </div>
</template>
<script>
export default {
  data() {
    return {
      //所有角色列表数据
      rolelist: [],
      // 控制分配权限的显示隐藏
      setRightDialogVisible:false,
           // 获取权限的数据
      rightslist:[],
      // 默认选中的节点的id值数组

      defKeys:[
        // 105,106
      ],
      // 当前即将分配权限的id
      roleId:'',
      // 树形控件的属性绑定对象
  treeProps:{
    label:'authName',
    children:'children'
  },
    //   控制添加用户对话框的显示与隐藏
    addDialogVisible:false,
    // 添加表单数据
    addFrom:{
        roleName:"",
        roleDesc:""
    },
    // 添加表单的验证规则
    addFromRules:{
         roleName: [
          { required: true,message: "请输入用户名", trigger: "blur" },
          {
            min: 2,
            max: 10,
            message: "用户名的长度在3~10个字符之间",
            trigger: "blur"
          }
        ]
        },
        // 控制修改用户对话框的显示与隐藏====编辑部分
    editDialogVisible:false,
        // 查询到的用户信息
      editForm:{
         roleName:"",
        roleDesc:""
      },
        // 修改表单的验证规则对象
        editFormRules:{
           roleName:{required:true,message:'请输入用户名',trigger:'blur'},
        roleDesc:{required:true,message:'请输入角色描述',trigger:'blur'}
      
        }
    }
  },
  created() {
    this.getRolesList();
  },
  methods: {
    // 获取所有角色的列表
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败");
      }
      this.rolelist = res.data;
      // console.log(this.rolelist);
    },
    //   监听添加用户对话框的关闭事件
    addDialogClosed() {
      // this.addDialogVisible=false
      this.$refs.addFormRef.resetFields();
    },
// 点击按钮 ，添加新用户
 addUser(){
    this.$refs.addFormRef.validate (async valid=>{
      // console.log(valid);true
        if(!valid) return
        // 可以发送添加用户的请求
      const {data:res} =await this.$http.post('roles',this.addFrom)
        if(res.meta.status !==200){
       this.$message.error('添加用户失败')
   }
   this.$message.success('添加用户成功！')
  //    隐藏添加用户的对话框
  //  console.log(res);
   this.addDialogVisible = false
// 重新获取用户列表的数据
this.getRolesList()
   })
},
// 展示编辑用户的对话框===============编辑
 async showEditDialog(id) {
      // console.log(id);
     const {data:res}=await this.$http.get('roles/'+id)
     if(res.meta.status !==200){
       return this.$message.error('查询用户信息失败')
     }
     this.editForm = res.data
       this.editDialogVisible=true
    },
    // 监听修改用户对话框的关闭事件
    editDialogClosed(){
      this.$refs.editFormRef.resetFields()
    },
    // 修改用户并提交
    editUserInfo(){
      this.$refs.editFormRef.validate(async valid=>{
        // console.log(valid);
        if(!valid) return
        // 发送修改用户信息的请求数据
      const {data:res} =await this.$http.put('roles/'+this.editForm.id,{
          email:this.editForm.email,
          mobile:this.editForm.mobile
        })
        console.log(this.editForm.email,this.editForm.mobile);
        if(res.meta.status !==200){
          return this.$message.error('更新用户信息失败')
        }
        // 关闭对话框
        this.editDialogVisible =false
        // 刷新数据列表
        this.getRolesList()
        // 提示修改成功
        this.$message.success('更新用户信息成功')
      })
    },

// async showEditDialog(id){
//   // console.log(id);
//  const {data:res} = await this.$http.get('roles/'+id)
//  if(res.meta.status !==200){
//    return this.$message.error('查询用户信息失败')
//  }
//  this.editForm = res.data
// this.editDialogVisible=true
// },
// editDialogClosed(){
//   this.$refs.editFormRef.resetFields()
// },
// // 修改用户信息并提交
// editUserInfo(){
// this.$refs.editFormRef.validate(async valid=>{
//   // console.log(valid);
//   if(!valid) return
//   // 发送修改用户信息的数据请求
// const {data:res}= await this.$http.put('roles/'+this.editForm.id,{
//     roleName:this.editForm.roleName,
//     roleDesc:this.editForm.roleDesc
//   })
//   console.log(this.editForm.roleName,this.editForm.roleDesc);
  
//   if(res.meta.status !==200){
//       console.log(res);
//     return this.$message.error('更新用户信息失败！')
//   }
//   // 关闭对话框
//   this.editDialogVisible =false
//   // 刷新数据
//   this.getRolesList()
//   // 显示修改成功
//   this.$message.success('更新用户修改成功')
// })
// },
   async removeUserById(id){
      console.log(id);
      // 弹框询问用户是否删除数据信息
     const confirmResult =await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        // 如果用户删除 则返回字符串 confirm
        // console.log(confirmResult);
        if(confirmResult!=='confirm'){
          return this.$message.info('已经取消删除')
        }
      const {data:res} = await this.$http.delete('roles/'+id)
      if(res.meta.status !==200){
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功')
      this.getRolesList();
    },
   async removeRightByID(role,rightId){
       const confirmResult=await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err=>err)
        if(confirmResult!=='confirm'){
          return this.$message.info('已经取消删除')
        }
        const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
        if(res.meta.status !==200){
          return this.$message.error('删除权限失败')
        }
        this.$message.success('删除权限成功')
        role.children = res.data
    },
    // 分配权限对话框
   async showSetRightDialog(role){
     this.roleId = role.id
// 获取所有数据
    const {data:res} = await this.$http.get('rights/tree')
    if(res.meta.status!==200){
      return this.$message.error('获取权限数据失败')
    }
    // 把获取到的权限数据保留到data中
    this.rightslist = res.data

    console.log(this.rightslist);
    // 递归获取三级节点的ID
    this.getLeafKeys(role,this.defKeys)

      this.setRightDialogVisible=true
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到defKeys数组中
    getLeafKeys(node, arr){
      if(!node.children){
        // 如果node节点不包含children属性 则是三级节点
        return arr.push(node.id)
      }
      node.children.forEach(item=>
      this.getLeafKeys(item,arr))
      
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed(){
      this.defKeys=[]
    },
  // 点击角色分配权限
 async allotRights(){
    const keys = [
      ...this.$refs.treeRef.getCheckedKeys(),
      ...this.$refs.treeRef.getHalfCheckedKeys()
    ]
    // console.log(keys);
    const idStr = keys.join(',')
    // console.log(idStr);
    
  const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
    if(res.meta.status !==200){
      return this.$message.error('分配权限失败')
    }
    console.log(res);
    
    this.$message.success('分配权限成功')
    this.getRolesList()
    this.setRightDialogVisible=false
  }
  }
};
</script>
<style lang="less" scoped>
.el-tag {
  margin: 10px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter{
  display: flex;
  align-items: center;
}
</style>