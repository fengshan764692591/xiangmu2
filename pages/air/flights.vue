<template>
  <section class="contianer">
    <el-row type="flex" justify="space-between">
      <div class="flights-content">
           <!-- 顶部过滤列表 -->
            <FlightsFilters :data="flightsData" />
        <!-- 航班头部布局 -->
        <FlightsListHead />
        <!--  其他代码... -->
        <!-- 航班列表 -->
        <FlightsItem v-for="(item,index) in dataList" :key="index" :data="item" />
        <!-- 分页  -->
        <el-row>
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="pageIndex"
            :page-sizes="[1, 2, 3, 4]"
            :page-size="pageIndex"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total"
          ></el-pagination>
        </el-row>
      </div>

      <!--  其他代码... -->
    </el-row>
  </section>
</template>

<script>
import FlightsListHead from "@/components/air/flightsListHead.vue";
import FlightsItem from "@/components/air/flightsItem.vue";
import FlightsFilters from "@/components/air/flightsFilters.vue";

export default {
  components: {
    FlightsListHead,
    FlightsItem,
    FlightsFilters
  },

  data() {
    return {
      flightsData: {
        flights: [],
        info:{},
        options: {}
      }, //获取总数据
      dataList: [],
      pageIndex: 1,
      pageSize: 5,
      total: 0
    };
  },
  mounted() {
    // 获取航班总数据
    this.$axios({
      url: "/airs",
      params: this.$route.query
    }).then(res => {
      console.log(res.data);
      this.flightsData = res.data;
      this.setDataList();
      this.total = res.total;
    });
  },
  methods: {
    // 分页
    handleSizeChange(val) {
      // console.log(`每页 ${val} 条`);
      this.pageSize = val;
      this.setDataList();
    },
    handleCurrentChange(val) {
      // console.log(`当前页: ${val}`);
      this.pageIndex = val;
      this.setDataList();
    },
    // 设置当前页的数据
    setDataList() {
      const start = (this.pageIndex - 1) * this.pageSize;
      const end = start + this.pageSize;
      this.dataList = this.flightsData.flights.slice(start, end);
    }
  }
};
</script>

<style scoped lang="less">
.contianer {
  width: 1000px;
  margin: 20px auto;
}

.flights-content {
  width: 745px;
  font-size: 14px;
}

.aside {
  width: 240px;
}
</style>