<template>
  <section class="contianer">
    <el-row type="flex" justify="space-between">
      <div class="flights-content">
        <!-- 顶部过滤列表 -->
        <!-- setDataList修改后过滤的数组列表 -->
        <FlightsFilters :data="cacheFlightsData" @setDataList="setDataList" />
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
        <!-- 侧边栏 -->
        <div class="aside">
          <!-- 侧边栏组件 -->
          <FlightsAside />
        </div>
      </div>

      <!--  其他代码... -->
    </el-row>
  </section>
</template>

<script>
import FlightsListHead from "@/components/air/flightsListHead.vue";
import FlightsItem from "@/components/air/flightsItem.vue";
import FlightsFilters from "@/components/air/flightsFilters.vue";
import FlightsAside from "@/components/air/flightsAside.vue";
export default {
  components: {
    FlightsListHead,
    FlightsItem,
    FlightsFilters,
    FlightsAside
  },

  data() {
    return {
      cacheFlightsData: {
        // 缓存一份数据，只会修改一次
        flights: [],
        info: {},
        options: {}
      },
      flightsData: {
        flights: [],
        info: {},
        options: {}
      }, //获取总数据
      dataList: [],
      pageIndex: 1,
      pageSize: 5,
      total: 0
    };
  },
  mounted() {
    // 获取机票列表数据
    this.getData()
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
    setDataList(arr) {
      // arr是展示的新的数据，该方法将会传替给过滤组件使用
      //  如果有数据就重第一页开始展示
      if (arr) {
        this.pageIndex = 1;
        this.flightsData.flights = arr;
        this.flightsData.total = arr.length;
      }
      const start = (this.pageIndex - 1) * this.pageSize;
      const end = start + this.pageSize;
      this.dataList = this.flightsData.flights.slice(start, end);
    },
    getData() {
      this.pageIndex = 1
      // 获取航班总数据
      this.$axios({
        url: "/airs",
        params: this.$route.query
      }).then(res => {
        console.log(res.data);
        this.flightsData = res.data;
        // 缓存一份数据，这个列表不会被修改
        // 而flightsData会被修改
        this.cacheFlightsData = { ...res.data };
        this.setDataList();
        this.total = res.total;
      });
    }
  },
  // 监听路由的变化
  watch: {
    $route() {
      // console.log(this.$route.query,33)
      this.getData()
    }
  }
};
</script>

<style scoped lang="less">
.contianer {
  width: 1000px;
  margin: 20px auto;
  position: relative;
}

.flights-content {
  width: 745px;
  font-size: 14px;
}

.aside {
  width: 240px;
  position: absolute;
  top: 43px;
  left: 400px;
}
</style>