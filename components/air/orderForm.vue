<template>
  <div class="main">
    <div class="air-column">
      <h2>剩机人</h2>
      <el-form class="member-info">
        <div class="member-info-item" v-for="(item,index) in users" :key="index">
          <el-form-item label="乘机人类型">
            <el-input placeholder="姓名" class="input-with-select" v-model="item.username">
              <el-select slot="prepend" value="1" placeholder="请选择">
                <el-option label="成人" value="1"></el-option>
              </el-select>
            </el-input>
          </el-form-item>

          <el-form-item label="证件类型">
            <el-input placeholder="证件号码" class="input-with-select" v-model="item.id">
              <el-select slot="prepend" value="1" placeholder="请选择">
                <el-option label="身份证" value="1" :checked="true"></el-option>
              </el-select>
            </el-input>
          </el-form-item>

          <span class="delete-user" @click="handleDeleteUser(index)">-</span>
        </div>
      </el-form>

      <el-button class="add-member" type="primary" @click="handleAddUsers">添加乘机人</el-button>
    </div>

    <div class="air-column">
      <h2>保险</h2>
      <div v-for="(item,index) in data.insurances" :key="index">
        <div class="insurance-item">
          <el-checkbox
            :label="`${item.type}: ￥${item.price}/份*${users.length} 最高赔付${item.compensation}`"
            border
            @change="handleInsurance(item)"
          ></el-checkbox>
        </div>
      </div>
    </div>

    <div class="air-column">
      <h2>联系人</h2>
      <div class="contact">
        <el-form label-width="60px">
          <el-form-item label="姓名">
            <el-input v-model="contactName"></el-input>
          </el-form-item>

          <el-form-item label="手机">
            <el-input placeholder="请输入内容" v-model="contactPhone">
              <template slot="append">
                <el-button @click="handleSendCaptcha">发送验证码</el-button>
              </template>
            </el-input>
          </el-form-item>

          <el-form-item label="验证码">
            <el-input v-model="captcha"></el-input>
          </el-form-item>
        </el-form>
        <el-button type="warning" class="submit" @click="handleSubmit">提交订单</el-button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    data: {
      type: Object,
      default: {}
    }
  },

  data() {
    return {
      users: [
        {
          username: "",
          id: ""
        }
      ],
      insurances: [], //保险数据
      contactName: "", //联系人的名字
      contactPhone: "", // 联系人的电话
      captcha: "000000", // 验证码
      invoice: false, // 发票
      seat_xid: "",
      air: ""
    };
  },
  methods: {
    // 添加乘机人
    handleAddUsers() {
      this.users = [...this.users, { username: "", id: "" }];
      // 方法2
      // this.users.push({username:'',id})
    },

    // 移除乘机人
    handleDeleteUser(index) {
      this.users.splice(index, 1);
    },

    // 发送手机验证码
    handleSendCaptcha() {
      this.$store.dispatch("user/sendCode", this.contactPhone).then(code => {
        this.$alert(`手机验证码是：${code}`, "提示", {
          confirmButtonText: "确定",
          type: "warning"
        });
      });
    },

    // 提交订单
    handleSubmit() {
      // 验证非空
      const rules = {
        users: {
          value: this.users[0].username && this.users[0].id,
          message: "乘机人不能为空"
        },
        contactName: { value: this.contactName, message: "联系人不能为空" },
        contactPhone: { value: this.contactName, message: "电话不能为空" }
        //   captcha:{value:this.contactName,
        //  message:'验证不能为空'
        //  }
      };
      let valid = true;
      Object.keys(rules).forEach(v => {
        if (!valid) return;
        // 如果为fales就返回错误信息
        if (!rules[v].value) {
          valid = false;
          this.$message.warning(rules[v].message);
        }
      });
      if (!valid) return;

      const data = {
        users: this.users,
        insurances: this.insurances,
        contactName: this.contactName,
        contactPhone: this.contactPhone,
        invoice: false,
        captcha: this.captcha,

        seat_xid: this.$route.query.seat_xid,
        air: this.$route.query.id
      };

      // 提交订单
      this.$axios({
        url: "/airorders",
        method: "POST",
        data,
        headers: {
          // token规范来自于JWT
          Authorization: `Bearer ${this.$store.state.user.userInfo.token}`
        }
      }).then(res => {
        console.log(res, 555);
        const { message, data } = res.data;

        this.$message.success(message);

        //  跳转到付款页
        this.$router.push("/air/pay?id=" + data.id);
      });
    },
    handleInsurance(item) {
      const index = this.insurances.indexOf(item.id);
      if (index > -1) {
        this.insurances.splice(index, 1);
      } else {
        this.insurances.push(item.id);
      }
      // console.log(this.insurances);
    }
  }
};
</script>

<style scoped lang="less">
.air-column {
  border-bottom: 1px #ddd dashed;
  padding-bottom: 20px;
  margin-bottom: 20px;
}

.air-column h2 {
  margin-bottom: 20px;
  font-size: 22px;
  font-weight: normal;
}

/deep/ .el-select .el-input {
  width: 130px;
}

.input-with-select {
  width: 590px;
}

.input-with-select /deep/ .el-input-group__prepend {
  background-color: #fff;
}
.member-info /deep/ .el-form-item {
  margin-bottom: 0;
}

.member-info-item {
  border-bottom: 1px #eee dashed;
  padding-bottom: 20px;
  position: relative;

  &:first-child {
    .delete-user {
      display: none;
    }
  }
}

.add-member {
  margin-top: 20px;
}

.delete-user {
  display: block;
  background: #ddd;
  width: 16px;
  height: 16px;
  font-size: 14px;
  text-align: center;
  line-height: 16px;
  color: #fff;
  cursor: pointer;
  border-radius: 50px;
  position: absolute;
  right: -30px;
  top: 50%;
}

.insurance {
  > div {
    margin-top: 10px;
  }
}

.insurance-item {
  margin-bottom: 20px;
}

.contact {
  /deep/ .el-input {
    width: 50%;
  }
}

.submit {
  margin: 50px auto;
  display: block;
  width: 250px;
  height: 50px;
}
</style>