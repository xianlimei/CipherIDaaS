<template>
  <div class="formed-wrap">
    <div class="formed-header">
      <div class="formed-header-title">{{showData.devicename}}</div>
      <div class="formed-header-btn">
        <myButton btnType="info"
                  v-if="!isEditStatus"
                  @click="edit">编辑</myButton>
      </div>
    </div>
    <div class="formed-container">
      <div class="formed-form"
           v-if="!isEditStatus">
        <p class="formed-form-item">
          <span>设备名称：</span>
          <span> {{showData.devicename}}</span>
        </p>
        <p class="formed-form-item">
          <span>设备地址：</span>
          <span> {{showData.equipAddr}}</span>
        </p>
        <p class="formed-form-item">
          <span>描述：</span>
          <span> {{showData.describee}}</span>
        </p>
        <p class="formed-form-item">
          <span>状态：</span>
          <span v-if="showData.status===0">
            <Badge status="success" />已启用</span>
          <span v-else>
            <Badge status="error" />已停用
          </span>
        </p>
        <p class="formed-form-item">
          <span>认证源：</span>
          <span> {{getAuthOrigin}}</span>
        </p>
        <p class="formed-form-item">
          <span>认证方式：</span>
          <span> {{getauthType}}</span>
        </p>
        <p class="formed-form-item">
          <span>二次认证：</span>
          <span> {{showData.towAuthWay===0?"钉钉Push认证":"无"}}</span>
        </p>
        <p class="formed-form-item">
          <span>Radius服务器：</span>
          <span> {{showData.radiusServer}}</span>
        </p>
        <p class="formed-form-item">
          <span>认证端口：</span>
          <span> {{showData.authPort}}</span>
        </p>
        <p class="formed-form-item">
          <span>shareKey：</span>
          <span> {{showData.shareKey}}</span>
        </p>
      </div>

      <Form v-else
            class="formed-form"
            :model="formItem"
            :label-width="120"
            :rules="ruleValidate"
            ref="formValidate">
        <FormItem label="设备名称："
                  prop="devicename">
          <Input v-model.trim="formItem.devicename"
                 placeholder="请输入" />
        </FormItem>
        <FormItem label="设备地址："
                  prop="equipAddr">
          <Input v-model.trim="formItem.equipAddr"
                 placeholder="请输入" /></Input>
        </FormItem>
        <FormItem label="描述："
                  prop="describee">
          <Input v-model.trim="formItem.describee"
                 placeholder="请输入"
                 type="textarea"
                 :rows="4"></Input>
        </FormItem>
        <FormItem label="状态："
                  prop="status">
          <RadioGroup v-model="formItem.status">
            <Radio v-for="item in statusList"
                   :label="item.value"
                   :key="item.value">{{item.label}}</Radio>
          </RadioGroup>
        </FormItem>
        <FormItem label="认证源："
                  prop="authOrigin">
          <Select v-model="formItem.authOrigin">
            <Option v-for="item in authOriginList"
                    :value="item.value"
                    :key="item.value">{{ item.label }}</Option>
          </Select>
        </FormItem>
        <FormItem label="认证方式："
                  prop="authType">
          <Select v-model="formItem.authType">
            <Option v-for="item in authTypeList"
                    :value="item.value"
                    :key="item.value">{{ item.label }}</Option>
          </Select>
        </FormItem>
        <FormItem label="二次认证："
                  prop="towAuthWay">
          <Checkbox v-model="formItem.towAuthWay">钉钉Push认证</Checkbox>
        </FormItem>
        <FormItem label="Radius服务器："
                  prop="radiusServer">
          <Input v-model.trim="formItem.radiusServer"
                 placeholder="请输入" />
        </FormItem>
        <FormItem label="认证端口："
                  prop="authPort">
          <Input v-model.trim="formItem.authPort"
                 placeholder="请输入"
                 number /></Input>
        </FormItem>
        <FormItem label="shareKey："
                  prop="shareKey">
          <Input v-model.trim="formItem.shareKey"
                 placeholder="请输入" /></Input>
        </FormItem>
      </Form>
      <div class="formed-footer"
           v-if="isEditStatus">
        <myButton btnType="info"
                  @click="handleSubmit('formValidate')">保存</myButton>
        <myButton @click="cancel">取消</myButton>
      </div>
    </div>
  </div>
</template>

<script>
import verify from "@/util/verify.js";
export default {
  data () {
    // 设备名称检验
    const validateName = (rule, value, callback) => {
      if (value.length > 15) {
        callback(new Error("请输入长度少于15个字的名称"));
      }
      callback();
    };
    // IP地址校验
    const validateIPAddress = (rule, value, callback) => {
      if (value.length > 0) {
        let IPv4 = verify.IPv4;
        let IPv6 = verify.IPv6;
        if (!IPv4.test(value) && !IPv6.test(value) && value !== "") {
          callback(new Error("IP格式不正确"));
        }
      }
      callback();
    };
    // 描述校验
    const validateDescription = (rule, value, callback) => {
      if (value && value.length > 50) {
        callback(new Error("请输入长度少于50个字的描述"));
      }
      callback();
    };
    return {
      isEditStatus: false, // 是否编辑状态
      uuid: "", // 设备id
      showData: {}, // 显示的数据
      formItem: {
        devicename: "", // 设备名称
        equipAddr: "", // 地址
        describee: "", // 描述
        status: 0, // 状态
        authOrigin: 0, // 认证源
        authType: 1, // 认证方式
        towAuthWay: true, // 二次认证
        radiusServer: "", // radius服务器
        authPort: "", // 认证端口
        shareKey: "" // shareKey
      },
      ruleValidate: {
        devicename: [
          { required: true, message: "设备名不能为空", trigger: "blur" },
          { validator: validateName, trigger: "blur" }
        ],
        equipAddr: [
          // { required: true, message: "地址不能为空", trigger: "blur" },
          { validator: validateIPAddress, trigger: "blur" }
        ],
        describee: [{ validator: validateDescription, trigger: "blur" }],
        radiusServer: [{ required: true, message: "radius服务器不能为空", trigger: "blur" }],
        authPort: [{ required: true, type: "number", message: "认证端口不能为空", trigger: "blur" }],
        shareKey: [{ required: true, message: "shareKey不能为空", trigger: "blur" }]
      },
      statusList: [ // 状态列表
        { value: 0, label: "启用" },
        { value: 1, label: "停用" }
      ],
      authOriginList: [ // 认证源选择列表
        { value: 0, label: "本地认证" },
        { value: 1, label: "AD认证" },
        { value: 2, label: "JLT" }
      ],
      authTypeList: [ // 认证方式列表
        { value: 1, label: "用户名+密码+动态密码" },
        { value: 2, label: "用户名+密码" },
        { value: 3, label: "用户名+动态密码" }
      ]

    };
  },
  computed: {
    getAuthOrigin () { // 计算认证源对应列表label
      let index = this.showData.authOrigin;
      if (typeof (index) === "undefined") return;
      return this.authOriginList[index].label;
    },
    getauthType () { // 计算认证源对应列表label
      if (typeof (this.showData.authType) === "undefined") return;
      let index = this.showData.authType - 1;
      return this.authTypeList[index].label;
    }
  },
  created () {
    this.uuid = this.$route.query.uuid;
    this.getData(); // 获取回显数据
  },
  methods: {
    /**
     * 获取设备回显数据
     * @param {*void}
     * @author yezi 2019-08-06
     */
    getData () {
      // 调用接口
      this.axios({
        method: "post",
        url: "/cipher/equip/details",
        data: { uuid: this.uuid }
      })
        .then(res => {
          if (res.data.return_code === 0) {
            this.showData = JSON.parse(JSON.stringify(res.data.return_result));
            this.formItem = JSON.parse(JSON.stringify(res.data.return_result));
            this.formItem.towAuthWay = this.formItem.towAuthWay === 0; // 二次认证转成boolen
          } else {
            this.$myMessage.error(res.data.return_msg);
          }
        })
        .catch(error => {
          this.$Notice.warning({
            title: "网络未知错误！",
            desc: error
          });
        });
    },
    /**
    * 点击编辑，进入编辑状态
    * @param {*void}
    * @author yezi 2019-08-07
    */
    edit () {
      this.isEditStatus = true;
    },
    /**
     * 点击取消，进入展示信息状态，并重置表单
     * @param {*void}
     * @author yezi 2019-08-07
     */
    cancel () {
      this.isEditStatus = false;
      this.$refs["formValidate"].resetFields(); // 取消后，对表单进行重置，恢复修改前状态
    },
    /**
     * 点击保存执行，表单验证通过后，调用保存函数
     * @param {*String iview表单ref名称，表单验证所需} name
     * @author yezi 2019-08-07
     */
    handleSubmit (name) {
      this.$refs[name].validate(valid => {
        if (valid) {
          this.saveData();
        } else {
          this.$myMessage.error("表单验证失败，请重试");
        }
      });
    },
    /**
     * 保存编辑数据，发送请求
     * @param {*void}
     * @author yezi 2019-08-02
     */
    saveData () {
      let params = {
        uuid: this.uuid,
        devicename: this.formItem.devicename,
        equipAddr: this.formItem.equipAddr,
        describee: this.formItem.describee,
        status: this.formItem.status,
        authOrigin: this.formItem.authOrigin,
        authType: this.formItem.authType,
        towAuthWay: this.formItem.towAuthWay ? 0 : 1,
        radiusServer: this.formItem.radiusServer,
        authPort: this.formItem.authPort,
        shareKey: this.formItem.shareKey
      };
      // 调用接口
      this.axios({
        method: "post",
        url: "/cipher/equip/compile",
        data: params
      })
        .then(res => {
          if (res.data.return_code === 0) {
            this.$myMessage.success(res.data.return_msg);
            this.getData();
            // 成功后 取消编辑状态，进入展示页
            setTimeout(() => {
              this.isEditStatus = false;
            }, 1000);
          } else {
            this.$myMessage.error(res.data.return_msg);
          }
        })
        .catch(error => {
          this.$Notice.warning({
            title: "网络未知错误！",
            desc: error
          });
        });
    }
  }
};
</script>

<style lang="less" scoped>
@import "~@/assets/styles/formStyle.less";
</style>
