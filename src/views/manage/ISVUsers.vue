<template>
  <section>
		<el-row>
			<el-col :span="20">
				<el-button size="medium" type="primary" @click="handleNewUser" v-if="isEditorUser&&NotApproStatus" >Add User</el-button>
        <el-button size="medium" type="primary" @click="handleDisableUser" v-if="isEditorUser&&NotApproStatus" >Disable All Users</el-button>
        <p/>
				<el-table :data="users">
          <!-- <el-table-column prop="username" label="Username" sortable></el-table-column> -->
          <el-table-column prop="email" label="Email" sortable></el-table-column>
          <el-table-column prop="realName" label="Name" sortable></el-table-column>
          <el-table-column label="Role">
            <template slot-scope="scope">
              {{scope.row.roles.map(item=>item.name).join(',')}}
            </template>
          </el-table-column>
          <el-table-column label="Status">
            <template slot-scope="scope">
              <el-switch v-model="scope.row.status" :active-value="1" :inactive-value="0" @change="handleStatusChange(scope.row)" :disabled="!isEditorUser"></el-switch>
            </template>
          </el-table-column>
          <!-- <el-table-column prop="" label="Last Login" :formatter="formatDate" sortable></el-table-column> -->
          <el-table-column label="Operation">
            <template slot-scope="scope">
              <el-button @click="handleEdit(scope.row)" type="plain" size="small" v-if="isEditorUser">Edit</el-button>
            </template>
          </el-table-column>
        </el-table>
        <p/>
        <el-pagination
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-size="pageSize"
          :total="total"
          layout="total, prev, pager, next">
        </el-pagination>

        <el-dialog  :visible.sync="dialogVisible" :open="handleOpen" :title="isNew?'Add User':'Edit User'" width="60%">
          <el-form :rules="rules2" v-if="form" ref="form" :key="formId" :model="form" label-width="120px" >
            <!-- <el-form-item label="Username" prop="username" v-if="isNew" :rules="[{ required: true, message:'Username is required', trigger: ['blur','change'] },{min:5, max:20, message:'The length must between 5 and 20 characters', trigger:['blur','change']},  { validator: validateUsername, trigger: ['blur'] }]">
              <el-input v-model="form.username"></el-input>
            </el-form-item>
            <el-form-item label="Username" prop="username" v-else required>
              <el-input v-model="form.username" disabled></el-input>
            </el-form-item> -->
            <el-form-item label="Email" prop="email" >
              <el-input v-model="form.email" :disabled="isNew?false:true"></el-input>
            </el-form-item>
            <el-form-item label="Name" prop="realName" :rules="[{required: true,message:'Name is required', trigger:'blur'},{max:50, message:'The length must be less than 50 characters', trigger:'blur'}]">
              <el-input v-model="form.realName"></el-input>
            </el-form-item>
            
            <!-- <el-form-item label="Email" prop="email" v-else >
              <el-input v-model="form.email" ></el-input>
            </el-form-item> -->
            <el-form-item label="Phone" prop="phone" >
              <el-input v-model="form.phone"  placeholder="Phone"  @keyup.native="handlePhoneChangeEvent($event)"  ></el-input>
            </el-form-item>
            <el-form-item label="Roles" prop="roleId" :rules="[{required: true,message:'Role is required',trigger:'blur'}]">
              <el-select v-model="form.roleId">
                <el-option v-for="item in isvRoleList" :key="item.id" :label="item.name" :value="item.id"></el-option>
              </el-select>
            </el-form-item>
          </el-form>
          <span slot="footer">
            <el-button @click="dialogVisible = false">Cancel</el-button>
            <el-button type="primary" @click="onSubmit">Confirm</el-button>
          </span>
        </el-dialog>
			</el-col>
		</el-row>
  </section>
</template>

<script>
import {
  userListUsingGET,
  existsUsersUsingGET,
  updateUserUsingPUT,
  updateUserUsingPUT_1,
  saveUserUsingPOST,
  listRolesUsingGET,
  updateUserRolesUsingPUT,
  getUserDetailsUsingGET,
  listClientUsersUsingGET,
  getIsvUsingGET,
  getIsvByCodeUsingGET,
  disableUserByIsvIdUsingPUT
} from "@/api";
import { handlePhoneChangeEvent, phoneRegExp } from "@/utils";
export default {
  data() {
    var validateEmail = (rule, value, callback) => {
      if (value) {
        if (value == this.copyEmail) {
          callback();
        }
        if (value != this.copyEmail) {
          existsUsersUsingGET({
            email: value
          }).then(response => {
            let { code, data, message } = response.data;
            if (code != 0) {
              this.$message({
                message: message,
                type: "error"
              });
            } else {
              if (data) {
                callback(new Error("Email has already exsited"));
              } else {
                callback();
              }
            }
          });
        }
      }
    };
    var checkNumber = (rule, value, callback) => {
      var reg = phoneRegExp;

      // var reg = /^[\d]{1,}[\S]*$/;
      // if (this.phoneDelete) {
      //   if (!value) {
      //     return callback(new Error("Phone is required"));
      //   }
      //   if (value.length === 3 || value.length === 7) {
      //     this.form.phone += "-";
      //   }
      // }

      if (!reg.test(value)) {
        return callback(new Error("Invalid value"));
      } else {
        callback();
      }
    };
    return {
      rules2: {
        phone: [
          {
            validator: checkNumber,
            required: true,
            trigger: ["blur"]
          },
          {
            max: 13,
            message: "The length must be less than 13 characters.",
            trigger: ["blur"]
          }
        ],
        email:[
          {
            required: true,
            type: "email",
            message: "Email is required",
            trigger: "blur"
          },
          {
            max: 50,
            message: "The length must be less than 50 characters",
            trigger: "blur"
          },
          { validator: validateEmail, trigger: ["blur"] }
        ]
      },
      validateUsername: (rule, value, callback) => {
        if (value) {
          if (/\W+/.test(value)) {
            callback(new Error("Only alphanumeric charactors are allowed"));
          }
          existsUsersUsingGET({
            username: value
          }).then(response => {
            let { code, data, message } = response.data;
            if (code != 0) {
              this.$message({
                message: message,
                type: "error"
              });
            } else {
              if (data) {
                callback(new Error("Username has already exsited"));
              } else {
                callback();
              }
            }
          });
        }
      },

      dialogVisible: false,
      users: [],
      form: {},
      copyEmail: "",
      isvRoleList: [],
      loading: false,
      formId: 0,
      currentPage: 1,
      pageNum: 1,
      pageSize: 20,
      total: 0,
      phoneDelete: false, //只触发验证
      isEditorUser: false, //没有编辑 isv列表的权限
      NotApproStatus: false, //公司审核状态为0 :waiting和2:reject 按钮隐藏
      username: [], //保存Edit初始值
      artpkId:"",// 保存pkId
    };
  },
  computed: {
    isNew() {
      return !this.form.hasOwnProperty("id");
    }
  },
  methods: {
    handlePhoneChangeEvent: handlePhoneChangeEvent,
    handleStatusChange(user) {
      updateUserUsingPUT({ id: user.id, status: user.status }).then(
        response => {
          let { code, data, message } = response.data;
          if (code !== 0) {
            this.$message({
              message: message,
              type: "error"
            });
          } else {
            this.$message({
              message: "Success",
              type: "success"
            });
            user.status = user.status;
          }
        }
      );
    },

    handleOpen() {
      this.$nextTick(() => {
        this.$refs.form.resetFields();
      });
    },

    handleNewUser() {
      this.formId++;
      this.form = {};
      this.$set(this.form, "roleId", null);
      this.dialogVisible = true;
    },
    handleDisableUser(){
    let tempData = JSON.parse(sessionStorage.getItem("artpkId"))
      disableUserByIsvIdUsingPUT({isvId:tempData.pkId}).then(response =>{
        this.loading = false;
         let { code, data, message } = response.data;
        if (code !== 0) {
          this.$message({
            message: message,
            type: "error"
          });
        } else{
          this.$message({
              message: "Success",
              type: "success"
          });
          let newStatus = this.users.map(item =>item.status ==1)
          newStatus = 1
          this.getUsers()
        }
      })
    },
    handleEdit(user) {
      this.formId++;
      this.form = Object.assign({}, user);
      //加监听
      this.$set(
        this.form,
        "roleId",
        user.roles.length > 0 ? user.roles[0].id : null
      );
      this.copyEmail = this.form.email;
      this.dialogVisible = true;
    },

    getUserDetail(id) {
      this.form = null;
      this.loading = true;
      getUserDetailsUsingGET({ id: id }).then(response => {
        this.loading = false;
        let { code, data, message } = response.data;
        if (code !== 0) {
          this.$message({
            message: message,
            type: "error"
          });
        } else {
          this.form = data;
          this.copyEmail = data.email;
         
        }
      });
    },

    onSubmit() {
      this.loading = true;
      this.$refs.form.validate(valid => {
        if (valid) {
          if (this.isNew) {
            this.form.username=this.form.email;
            this.createUser();
          } else {
            this.updateUser();
          }
        }
      });
    },
    createUser() {
       let artpkId=JSON.parse(sessionStorage.getItem("artpkId"));
      this.form["clientId"] = artpkId.pkId;
      this.form["type"] = 2;
      this.form["gender"] = 0;
      this.form["address"] = "";
      this.form["avatarPath"] = "";
      this.form["position"] = "";
      saveUserUsingPOST({ saveUserReq: this.form }).then(response => {
        this.loading = false;
        let { code, data, message } = response.data;
        if (code !== 0) {
          this.$message({
            message: message,
            type: "error"
          });
        } else {
          this.form["id"] = data;
          updateUserRolesUsingPUT({
            userId: this.form.id,
            roleIds: [this.form.roleId]
          }).then(response => {
            this.loading = false;
            let { code, data, message } = response.data;
            if (code != 0) {
              this.$message({
                message: message,
                type: "error"
              });
            } else {
              this.$message({
                message: "Success",
                type: "success"
              });
              this.getUsers();
              this.dialogVisible = false;
            }
          });
        }
      });
    },

    updateUser() {
      updateUserUsingPUT_1({ id: this.form.id, updateUserReq: this.form }).then(
        response => {
          this.loading = false;
          let { code, data, message } = response.data;
          if (code !== 0) {
            this.$message({
              message: message,
              type: "error"
            });
          } else {
            updateUserRolesUsingPUT({
              userId: this.form.id,
              roleIds: [this.form.roleId]
            }).then(response => {
              this.loading = false;
              let { code, data, message } = response.data;
              if (code != 0) {
                this.$message({
                  message: message,
                  type: "error"
                });
              } else {
                this.$message({
                  message: "Success",
                  type: "success"
                });
                this.getUsers();
                this.dialogVisible = false;
              }
            });
          }
        }
      );
    },

    getUsers() {
      let artpkId=JSON.parse(sessionStorage.getItem("artpkId"));
      this.loading = true;
      userListUsingGET({
        type: 2,
        clientId: artpkId.pkId,
        pageNum: this.pageNum,
        pageSize: this.pageSize
      }).then(response => {
        this.loading = false;
        let { code, data, message } = response.data;
        if (code !== 0) {
          this.$message({
            message: message,
            type: "error"
          });
        } else {
          let realName = data.list.filter(u => u.realName);
          sessionStorage.setItem("realName", JSON.stringify(realName));
          this.users = data.list;
          this.total = data.total;
          this.currentPage = data.pageNum;
        }
      });
    },

    getRoles(type) {
      this.loading = true;
      listRolesUsingGET({ type: 2 }).then(response => {
        this.loading = false;
        let { code, data, message } = response.data;
        if (code !== 0) {
          this.$message({
            message: message,
            type: "error"
          });
        } else {
          this.isvRoleList = data;
        }
      });
    },
    handleCurrentChange(val) {
      this.pageNum = val;
      this.getUsers();
    },
    getIsvDetail() {
      this.loading = true;
      getIsvByCodeUsingGET({ code: this.$route.params.number }).then(response => {
        this.loading = false;
        let { code, data, message } = response.data;
        if (code !== 0) {
          this.$message({
            message: message,
            type: "error"
          });
        } else {
          if (data.ndaStatus == 0 || data.ndaStatus == 2) {
            this.NotApproStatus = false;
          } else {
            this.NotApproStatus = true;
          }
        }
      });
    }
  },
  mounted() {
    this.getUsers();
    this.getRoles();
  },
  created() {
    this.getIsvDetail();
    let user = JSON.parse(sessionStorage.getItem("user"));

    //没有编辑 isv列表的权限
    this.isEditorUser = user.authorities.filter(
      u => u.authority == "isv:Edit ISV Info:2"
    );
    if (this.isEditorUser.length) {
      this.isEditorUser = true;
    } else {
      this.isEditorUser = false;
    }
  }
};
</script>

<style lang="scss" scoped>
</style>
