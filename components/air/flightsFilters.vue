<template>
  <div class="filters">
    <el-row type="flex" class="filters-top" justify="space-between" align="middle">
      <el-col :span="8">
        单程：
        {{data.info.departCity}} - {{data.info.destCity}}
        /
        {{data.info.departDate}}
      </el-col>
      <el-col :span="4">
        <el-select size="mini" v-model="airport" placeholder="起飞机场" @change="handleAirport">
          <el-option
            label="白云机场"
            value="白云机场"
            v-for="(item,index) in data.options.airport"
            :key="index"
          ></el-option>
        </el-select>
      </el-col>
      <el-col :span="4">
        <!--  :label数据要动态绑定 -->
        <el-select size="mini" v-model="flightTimes" placeholder="起飞时间" @change="handleFlightTimes">
          <el-option
            :label="`${item.from}:00 -${item.to}:00`"
            :value="item"
            v-for="(item,index) in data.options.flightTimes"
            :key="index"
          ></el-option>
        </el-select>
      </el-col>
      <el-col :span="4">
        <el-select size="mini" v-model="company" placeholder="航空公司" @change="handleCompany">
          <el-option
            :label="item"
            :value="item"
            v-for="(item,index) in data.options.company"
            :key="index"
          ></el-option>
        </el-select>
      </el-col>
      <el-col :span="4">
        <el-select size="mini" v-model="airSize" placeholder="机型" @change="handleAirSize">
          <el-option
            :label="item.name"
            :value="item.value"
            v-for="(item,index) in optionSize"
            :key="index"
          ></el-option>
        </el-select>
      </el-col>
    </el-row>
    <div class="filter-cancel">
      筛选：
      <el-button type="primary" round plain size="mini" @click="handleFiltersCancel">撤销</el-button>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    data: {
      // 父元素传数据
      type: Object,
      default: {}
    }
  },
  data() {
    return {
      airport: "", // 机场
      flightTimes: "", // 出发时间
      company: "", // 航空公司
      airSize: "", // 机型大小
      optionSize: [
        { name: "大", value: "L" },
        { name: "中", value: "M" },
        { name: "小", value: "N" }
      ]
    };
  },
  methods: {
    // 选择机场时候触发,filter过滤
    handleAirport(value) {
      const arr = this.data.flights.filter(v => {
      return  v.org_airport_name === value;
      });
      this.$emit("setDataList", arr);
    },

    // 选择出发时间时候触发
    handleFlightTimes(value) {
      // console.log(value)
     const arr = this.data.flights
     .filter(v =>{
       return value.from <= +v.dep_time.split(":")[0] && value.to >= +v.dep_time.split(":")[0]
     })
      this.$emit("setDataList",arr)
    },

    // 选择航空公司时候触发
    handleCompany(value) {
      // console.log(value)
  const arr = this.data.flights.filter(v =>{
  return  v.airline_name === value
    // console.log(this.arr)
  })
  this.$emit("setDataList",arr)
     
    },

    // 选择机型时候触发
    handleAirSize(value) {
   const arr = this.data.flights.filter(v =>{
     return v.plane_size === value
   })
   this.$emit("setDataList",arr)
    },

    // 撤销条件时候触发
    handleFiltersCancel() {
      this.airport = "",
      this.flightTimes = "",
      this.company = "",
      this.optionSize =""
      this.$emit("setDataList",this.data.flights)
    }
  }
};
</script>

<style scoped lang="less">
.filters {
  margin-bottom: 20px;
}

.filters-top {
  > div {
    /deep/ .el-select {
      margin-left: 10px;
    }
  }
}

.filter-cancel {
  margin-top: 10px;
}
</style>