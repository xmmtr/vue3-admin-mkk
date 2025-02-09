<template>
  <el-card class="good-container">
    <template #header>
      <div class="header">
        <el-button type="primary" :icon="Plus" @click="handleAdd"
          >新增商品</el-button
        >
        <el-input v-model="keyword" class="w-50 m-2" :prefix-icon="Search" placeholder="请输入要搜索的内容" style="width: 250px;margin: 0 5px;" />
        <el-button type="primary" :icon="Search" @click="getGoodList(keyword)">搜索商品</el-button>
      </div>
    </template>
    <el-table
      :load="state.loading"
      :data="state.tableData"
      tooltip-effect="dark"
      style="width: 100%"
    >
      <el-table-column prop="id" label="商品编号"> </el-table-column>
      <el-table-column prop="commodityname" label="商品名"> </el-table-column>
      <el-table-column prop="commodityintro" label="商品简介">
      </el-table-column>
      <el-table-column label="商品图片" width="150px">
        <template #default="scope">
          <img
            style="width: 100px; height: 100px"
            :key="scope.row.goodsId"
            :src="$filters.prefix(scope.row.commodityimage)"
            alt="商品主图"
          />
        </template>
      </el-table-column>
      <el-table-column prop="weight" label="重量"> </el-table-column>
      <el-table-column prop="commoditymoney" label="商品售价">
      </el-table-column>
      <el-table-column prop="commodityType" label="商品类型"> </el-table-column>
      <el-table-column prop="flavor" label="商品口味"> </el-table-column>
      <el-table-column prop="material" label="商品原料"> </el-table-column>

      <el-table-column label="操作" width="100">
        <template #default="scope">
          <a
            style="cursor: pointer; margin-right: 10px"
            @click="handleEdit(scope.row.id)"
            >修改</a
          >
          <a
            style="cursor: pointer; margin-right: 10px"
            @click="handleDelete(scope.row.id)"
            >删除</a
          >
        </template>
      </el-table-column>
    </el-table>
    <!--总数超过一页，再展示分页器-->
    <el-pagination
      background
      layout="prev, pager, next"
      :total="state.total"
      :page-size="state.pageSize"
      :current-page="state.currentPage"
      @current-change="changePage"
    />
  </el-card>
</template>

<script setup>
import { onMounted, reactive, getCurrentInstance, ref } from "vue";
import axios from "@/utils/axios";
import { ElMessage } from "element-plus";
import { Plus, Delete } from "@element-plus/icons-vue";
import { useRouter } from "vue-router";
import { Search } from '@element-plus/icons-vue'

const app = getCurrentInstance();
const { goTop } = app.appContext.config.globalProperties;
const router = useRouter();
const state = reactive({
  loading: false,
  tableData: [], // 数据列表
  total: 0, // 总条数
  currentPage: 1, // 当前页
  pageSize: 10, // 分页大小
});
onMounted(() => {
  getGoodList(null);
});
const keyword = ref()


// 获取商品列表
const getGoodList = (keyword) => {
  state.loading = true;
  axios
    .get("/commodity/getPagecommodity", {
      params: {
        pageNum: state.currentPage,
        pageSize: state.pageSize,
        keyword: keyword?keyword:null
      },
    })
    .then((res) => {
      console.log(res)
      state.tableData = res.data;
      state.total = res.total
      state.currentPage = res.page
      state.loading = false;
      goTop && goTop();
    });
};
const handleDelete = (goodsId) => {
  state.loading = true;
  // 找到id的下标,删除该项
  const index = state.tableData.findIndex( e => goodsId === e.id)
  state.tableData.splice(index, 1)
  axios
    .delete("/commodity", {
      data: {
        // 请求参数放在请求体
        id: goodsId,
      },
    })
    .then((res) => {
      console.log(res);
      state.tableData.find
      console.log(state.tableData);
      // state.total = res.totalCount
      // state.currentPage = res.currPage
      state.loading = false;
      goTop && goTop();
    });
};
const handleAdd = () => {
  router.push({ path: "/add" });
};
const handleEdit = (id) => {
  router.push({ path: "/add", query: { id } });
};
const changePage = (val) => {
  state.currentPage = val;
  getGoodList(null);
};
</script>

<style scoped>
.good-container {
  min-height: 100%;
}
.el-card.is-always-shadow {
  min-height: 100% !important;
}
</style>
