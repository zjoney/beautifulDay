<template>
  <section>
    <bread-crumb>
    </bread-crumb>
    <el-row class="toolbar">
      <el-col :span="20">
        <el-form ref="form" :model="form" label-width="200px" :rules="rules2">
          <el-form-item label="Company Name" prop="companyName" >
            <el-input v-model="form.companyName"></el-input>
          </el-form-item>

          <el-form-item label="Address" required>
            <el-form-item prop="companyAddress" :rules="[{ required: true, message: 'Address is required', trigger: 'blur'},{ max: 255, message: 'The length must be less than 255 characters', trigger: 'blur' }]" >
              <el-input v-model="form.companyAddress" placeholder="Address"></el-input>
            </el-form-item>
            <p/>
            <el-col :span="7">   
              <el-form-item prop="postCode" :rules="[{required:true, message: 'Zip Code is required', trigger: 'blur'},{max: 50, message: 'The length must be less than 50 characters', trigger:'blur'}]">
                <el-input v-model="form.postCode" placeholder="Zip Code"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="1">&nbsp;</el-col>
            <el-col :span="7">
              <el-form-item prop="city" :rules="[{required:true, message:'City is required', trigger:'blur' },{max:50, message: 'The length must be less than 50 characters', trigger:'blur'}]">
                <el-input v-model="form.city" placeholder="City"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="1">&nbsp;</el-col>
            <el-col :span="7">
              <el-form-item prop="state" :rules="[{required:true, message:'State is required', trigger:'blur' },{max:50, message: 'The length must be less than 50 characters', trigger:'blur'}]">
                <el-input v-model="form.state" placeholder="State"></el-input>
              </el-form-item>
            </el-col>
          </el-form-item>
          
          <el-form-item label="Website" prop="companyWebsite" :rules="[{ max: 255, message: 'The length must be less than 255 characters', trigger: 'blur' }]">
            <el-input v-model="form.companyWebsite"></el-input>
          </el-form-item>

          <el-form-item label="Contact Person" required :class="{expand:isExpand}">
            <el-col :span="7">
              <el-form-item prop="businessContactName" :rules="[{required: true,message:'Name is required', trigger:'blur'},{max:50, message:'The length must be less than 50 characters', trigger:'blur'}]">
                <el-input v-model="form.businessContactName" placeholder="Name"></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="1">&nbsp;</el-col>
            <el-col :span="7">
              <el-form-item prop="businessContactPhone" >
                <el-input v-model="form.businessContactPhone" placeholder="Phone"  @keyup.native="handlePhoneChangeEvent($event)"  ></el-input>
              </el-form-item>
            </el-col>
            <el-col :span="1">&nbsp;</el-col>
            <el-col :span="7">
              <el-form-item prop="businessContactEmail">
                <el-input v-model="form.businessContactEmail" placeholder="Work Email"></el-input>
              </el-form-item>
            </el-col>
          </el-form-item>

          <el-form-item label="POS System Name" prop="posSystemName" :rules="[{ required: true, message: 'POS System Name is required', trigger: 'blur' },{max:255, message:'The length must be less than 255 characters', trigger:'blur'}]">
            <el-input v-model="form.posSystemName"></el-input>
          </el-form-item>

          <el-form-item label="POS System Version" prop="posSystemVersion" :rules="[{  message: 'POS System Version is required', trigger: 'blur' },{max:255, message:'The length must be less than 255 characters', trigger:'blur'}]">
            <el-input v-model="form.posSystemVersion"></el-input>
          </el-form-item>

          <el-form-item label="POS Type" prop="dictIds" :rules="[{required: true, message:'POS Type is required' ,trigger:'blur'}]">
            <el-checkbox-group v-model="form.dictIds">
              <el-checkbox v-for="item in posList" :key="item.pkId" :label="item.pkId">{{item.item}}</el-checkbox>
            </el-checkbox-group>
          </el-form-item>

          <el-form-item label="Interested Terminals" prop="dictIds2" :rules="[{required:true, message:'Interested Terminals is required',trigger:'blur'}]">
            <el-checkbox-group v-model="form.dictIds2">
              <el-checkbox v-for="item in modelList" :key="item.pkId" :label="item.pkId">{{item.item}}</el-checkbox>
            </el-checkbox-group>
          </el-form-item>
          
          <el-form-item label="Integration Type" prop="dictIds3" :rules="[{required:true, message:'Integration Type is required',trigger:'blur'}]">
              <el-checkbox-group v-model="form.dictIds3">
              <el-checkbox v-for="item in integraList" :label="item.pkId" :key="item.pkId">
              <el-tooltip  placement="bottom"  effect="light">
                <div slot="content" v-if="item.pkId==16">
                  The ECR/POS sends transaction data (transaction type, amount ...etc.) to the PAX terminal. <br/>The PAX terminal reads the card, transmits the secure data to the host, and passes the host<br/> response back to the ECR/POS without sensitive card data. The ECR/POS does not <br/>communicate with the host. In this scenario, the ECR/POS is out-out-PCI scope.
                </div>
                <div slot="content" v-else>
                  The PAX terminal is a peripheral device used to swipe, insert or tap a card and enter the PIN. <br/>It reads the card and sends all cardholder data encrypted back to the ECR/POS. The ECR/POS <br/>handles the payment logic and the communication with the host. In this scenario,<br/> the ECR/POS is required to support PCI requirements even when the card data is encrypted.
                </div>
                <el-button type="text" @click="onClickIntBtn(item.pkId)"> {{item.item}}</el-button>
              </el-tooltip>
              </el-checkbox>
              </el-checkbox-group>
          </el-form-item>

          <el-form-item label="NDA" prop="ndaFilePath" >
            <el-upload ref="upload" :action="uploadUrl" accept=".pdf" :show-file-list="true"  :on-success="handleUploadFile" :file-list="uploadFileList" :before-upload="handleBefore" :on-progress="uploadFileProcess">
               <el-button size="small" ><i class="el-icon-upload"></i>Upload</el-button>
            </el-upload>
            <transition name="fade">
               <el-progress v-if="fileFlag" type="line" :percentage="fileUploadPercent" status="success"></el-progress>
            </transition>
            <i style="color: #a4a2a2;font-size: 14px;">Support extension.pdf</i>
          </el-form-item>

          <el-form-item label="Signing Date" prop="signDate" :rules="[{required: true, message:'Signing Date is required' ,trigger:'blur'}]">
            <el-date-picker v-model="form.signDate" type="date" value-format="timestamp"></el-date-picker>
          </el-form-item>

          <el-form-item label="Sales" prop="salesId" :rules="[{required: true, message:'Sales is required' ,trigger:'blur'}]">
             <el-select v-model="form.salesId">
                <el-option v-for="item in SalesList" :key="item.id" :label="item.realName" :value="item.id"></el-option>
              </el-select>
          </el-form-item>

          <el-form-item label="SE" prop="salesEngineerId" :rules="[{required: true, message:'SE is required' ,trigger:'blur'}]">
            <el-select v-model="form.salesEngineerId" >
                <el-option v-for="item in SalesEngineerList" :key="item.id" :label="item.realName" :value="item.id"></el-option>
              </el-select>
          </el-form-item>

          <el-form-item label="Support" prop="supportId" :rules="[{required: true, message:'Support is required' ,trigger:'blur'}]">
            <el-select v-model="form.supportId">
                <el-option v-for="item in SupportList" :key="item.id" :label="item.realName" :value="item.id"></el-option>
              </el-select>
          </el-form-item>

          <el-form-item>
            <el-button @click="$router.push({path: '/manage/isvs'})">Cancel</el-button>
            <el-button type="primary" @click="onSubmit">Confirm</el-button>
          </el-form-item>
        </el-form>
      </el-col>
    </el-row>
  </section>
</template>

<script>
import dateFormat from "dateformat";
import {
  listIsvBaseInfoUsingGET,
  saveIsvUsingPOST,
  listIsvUsingGET,
  listRolesUsingGET,
  userByRoleUsingGET,
  getUserDetailsUsingGET,
  updateUserRolesUsingPUT,
  updateIsvUsingPUT,
  userListUsingGET,
  existsUsersUsingGET,
  checkExistUsingGET
} from "@/api";
import { handlePhoneChangeEvent,phoneRegExp } from "@/utils";
export default {
  data() {
    var checkNumber = (rule, value, callback) => {
     
      var reg=phoneRegExp;
      // var reg = /^[\d]{1,}[\S]*$/;
      // if (this.phoneDelete) {
      //   if (!value) {
      //     return callback(new Error("ContactPhone is required"));
      //   }

      //   if (value.length === 3 || value.length === 7) {
      //     //文本框输入
      //     this.form.businessContactPhone += "-";
      //   }
      // }
      if (!reg.test(value)) {
        return callback(new Error("Invalid value"));
      } else {
        callback();
      }
    };
    var validateEamil = (rule, value, callback) =>{
      var str = ["aol.com","att.net", "comcast.net", "facebook.com", "gmail.com", "gmx.com", "googlemail.com", "google.com","hotmail.com", "hotmail.co.uk", "mac.com", "me.com", "mail.com", "msn.com", "live.com", "sbcglobal.net", "verizon.net","yahoo.com", "yahoo.co.uk", "outlook.com", "sina.com", "sina.cn", "qq.com", "naver.com", "hanmail.net", "daum.net","nate.com", "yahoo.co.jp", "yahoo.co.kr", "yahoo.co.id", "yahoo.co.in", "yahoo.com.sg", "yahoo.com.ph", "163.com","126.com", "aliyun.com", "foxmail.com"]
      let _this = this;
      var isExpand=false;
      for(var i of Object.keys(str)){
        if(value.lastIndexOf(str[i]) !== -1){
           isExpand = true;
           callback(new Error('Please use work email to register. Using personal mail may result in your application being rejected.'))
        }
      }
      _this.isExpand=isExpand;
      if(value){
        callback();
      //   existsUsersUsingGET({email: value}).then(response => {
      //       let { code, data, message } = response.data;
      //       if (code != 0) {
      //       _this.$message({
      //         message: message,
      //         type: "error"
      //       });
      //       } else {
      //         if (data) {
      //           callback(new Error("Work Email has already exsited"));
      //         } else {
      //           callback();
      //         }
      //       }
      //   })
      }
    };
    var validateCompanyName = (rule, value, callback) => {
      //  console.log(value)
      if (value) {
        // if (/\W+/.test(value)) {
        //   callback(new Error("Only alphanumeric charactors are allowed"));
        // }
        checkExistUsingGET({
          companyName: value
        }).then(response => {
          let { code, data, message } = response.data;
          if (code != 0) {
            this.$message({
              message: message,
              type: "error"
            });
          } else {
            if (data) {
              callback(new Error("CompanyName has already exsited"));
            } else {
              callback();
            }
          }
        });
      }
    };
    return {
      isExpand: false,
      phoneDelete: false, //只触发验证
      uploadFileList:[],
      fileFlag: false,
      fileUploadPercent: 0,
      filePath: '',
      integraList: [],
      form: {
        companyName: "",
        companyAddress: "",
        city: "",
        state: "",
        postCode: "",
        companyWebsite: "",
        businessContactEmail: "",
        businessContactName: "",
        businessContactPhone: "",
        technicalContactEmail: "",
        technicalContactName: "",
        technicalContactPhone: "",
        dictIds: [],
        dictIds2: [],
        dictIds3: [],
        posSystemName: "",
        posSystemVersion: "",
        ndaFilePath: '',
        // uploadFileList: [],
        signDate: null,
        salesId: null,
        salesEngineerId: null,
        supportId: null
      },
      posList: [],
      modelList: [],
      paxUserList: [],
      SalesList: [],
      SalesEngineerList: [],
      SupportList: [],
      copyPhone: "", //保存初始的businessContactPhone
      rules2: {
        businessContactPhone: [
          {
            validator: checkNumber,
            required: true,
            trigger: ["blur"]
          },
          {
            max: 13,
            message: "Length exceeds 13",
            trigger: ["blur"]
          }
        ],
        companyName: [
          {
            required: true,
            message: "Company Name is required",
            trigger: "blur"
          },
          {
            max: 255,
            message: "The length must be less than 255 characters",
            trigger: "blur"
          },
          { validator: validateCompanyName, trigger: ["blur"] }
        ],
        businessContactEmail: [
          { 
            required: true, type: 'email', message: 'Work Email is required', trigger: 'blur' 
          },
          {
            max:50, message:'The length must be less than 50 characters', trigger:'blur'
            },
          { 
            validator: validateEamil, trigger:["blur"]
          }
        ]
      }
    };
  },
  computed :{
    uploadUrl () {
      return '/api/v1/files/upload?accessType=0&attachedType=3';
    }
  },
  watch: {
  },
  methods: {
    handlePhoneChangeEvent:handlePhoneChangeEvent,
    
    formatDate(timestamp) {
      if (timestamp != null) {
        return dateFormat(new Date(parseInt(timestamp)), "yyyy-mm-dd");
      } else {
        return "NA";
      }
    },
    handleBefore (file,fileList) {
        // console.log(file)
        if (file.name.lastIndexOf('.pdf') === -1) {
        this.$message.error('Sorry, only pdf are allowed');
        return false;
      }
      if (file.size === 0) {
        this.$message.error('Sorry, The file must not be empty.');
        return false;
      }
      if (file.size / 1024 / 1024 > 300) {
        this.$message.error('Sorry, No more than 300MB');
        return false;
      }
      this.uploadFileList = [file];
    },
    handleUploadFile (response,file,fileList) {
      if (file.name.lastIndexOf('.pdf') === -1) {
        this.$message.error('Not supported file type');
        this.uploadFileList = [];
        return false;
      }
      if (file.size === 146) {
        this.$message.error('Sorry, The file must not be empty.');
        this.uploadFileList = [];
        return false;
      }
      if (file.size / 1024 / 1024 > 300) {
        this.$message.error('File exceeds maximum size limit 300MB');
        this.uploadFileList = [];
        return false;
      }
      this.uploadFileList = [file];
      let {data} = response
    
      this.form.ndaFilePath = data.path;
      return true;
    },
    uploadFileProcess(event, file, fileList){
    this.fileFlag = true;
    this.fileUploadPercent = parseInt(event.percent);
    setTimeout(()=>{
      if(event.percent == 100){
        this.fileFlag = false;
      }
    },1000)
},

    getPaxUsers() {
      this.loading = true;
      userListUsingGET({ pageSize: 0, type: 1 }).then(response => {
        this.loading = false;
        let { code, data, message } = response.data;
        if (code != 0) {
          this.$message({
            message: message,
            type: "error"
          });
        } else {
          this.paxUserList = data.list;
          for (var i = 0; i < this.paxUserList.length; i++) {
            var nowItem = this.paxUserList[i];
            if (
              nowItem.roles.findIndex(item => {
                return item.id == 2;
              }) != -1
            ) {
              this.SupportList.push(Object.assign({}, nowItem));
            }
            if (
              nowItem.roles.findIndex(item => {
                return item.id == 3;
              }) != -1
            ) {
              this.SalesEngineerList.push(Object.assign({}, nowItem));
            }
            if (
              nowItem.roles.findIndex(item => {
                return item.id == 4;
              }) != -1
            ) {
              this.SalesList.push(Object.assign({}, nowItem));
            }
          }
        }
      });
    },
    getIsvDicts() {
      listIsvBaseInfoUsingGET().then(response => {
        let { code, data, message } = response.data;

        this.modelList = data.filter(
          item => item.group == "Payment Terminal Type"
        );
        let tempList = []
        let posAll = data.filter(
          item =>
            item.group === 'POS Basic Information' && item.subgroup === 'POS Type'&& item.item !=="Other"
        )
        let posNew =data.filter(
          item =>
            item.item =="Other"
        )
        this.posList = tempList.concat(posAll, posNew)

        this.integraList = data.filter( item => item.group == "POS Basic Information" && item.subgroup == "Integration Type")
      });
    },

    onSubmit() {
      this.loading = true;
      this.$refs.form.validate(valid => {
        if (valid) {
          this.form.dictIds = this.form.dictIds.concat(this.form.dictIds2,this.form.dictIds3);
          saveIsvUsingPOST({ req: this.form }).then(response => {
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
              this.$router.push("/manage/isvs");
            }
          });
        }
      });
    },
    onClickIntBtn(id){
      if(this.form.dictIds3&&this.form.dictIds3.indexOf(id)!=-1){
        var inx=this.form.dictIds3.indexOf(id);
        this.form.dictIds3.splice(inx,1);
      }else{
        this.form.dictIds3.push(id);
      }
    }
  },
  created() {
    this.getPaxUsers();
    this.getIsvDicts();
  }
};
</script>

<style scoped lang="scss">
.el-button--text{
  color:#606266;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
/deep/ .el-upload-list__item {
  .el-progress{
    display: none;
  }
}
.expand{
  margin-bottom: 60px;
}
</style>