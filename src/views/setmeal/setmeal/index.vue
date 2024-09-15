<template>
  <div class="app-container">
    <el-form
      :model="queryParams"
      ref="queryRef"
      :inline="true"
      v-show="showSearch"
      label-width="68px"
    >
      <el-form-item label="套餐分类" prop="categoryId">
        <el-input
          v-model="queryParams.categoryId"
          placeholder="请输入套餐分类"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="套餐名称" prop="name">
        <el-input
          v-model="queryParams.name"
          placeholder="请输入套餐名称"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="套餐价格" prop="price">
        <el-input
          v-model="queryParams.price"
          placeholder="请输入套餐价格"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="售卖状态" prop="status">
        <el-select
          v-model="queryParams.status"
          placeholder="请选择售卖状态"
          clearable
        >
          <el-option
            v-for="dict in seal_system"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery"
          >搜索</el-button
        >
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
      <el-col :span="1.5">
        <el-button plain type="primary" icon="Plus" @click="handleAdd" v-hasPermi="['setmeal:setmeal:add']">
          新增
        </el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['setmeal:setmeal:edit']"
          >修改</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['setmeal:setmeal:remove']"
          >删除</el-button
        >
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['setmeal:setmeal:export']"
          >导出</el-button
        >
      </el-col>
      <right-toolbar
        v-model:showSearch="showSearch"
        @queryTable="getList"
      ></right-toolbar>
    </el-row>

    <el-table
      v-loading="loading"
      :data="setmealList"
      @selection-change="handleSelectionChange"
    >
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="套餐名称" align="center" prop="name" />
      <el-table-column label="图片" align="center" prop="image" width="100">
        <template #default="scope">
          <image-preview :src="scope.row.image" :width="50" :height="50" />
        </template>
      </el-table-column>
      <el-table-column label="套餐分类" align="center" prop="categoryId">
        <template #default="scope">
          <span>{{ categoryNames[scope.row.categoryId] || "加载中..." }}</span>
        </template>
      </el-table-column>
      <el-table-column label="套餐价格" align="center" prop="price">
        <template #default="scope">
          <span>￥{{ scope.row.price }}</span>
        </template>
      </el-table-column>
      <el-table-column label="售卖状态" align="center" prop="status">
        <template #default="scope">
          <dict-tag :options="seal_system" :value="scope.row.status" />
        </template>
      </el-table-column>
      <el-table-column
        label="更新时间"
        align="center"
        prop="updateTime"
        width="180"
      >
        <template #default="scope">
          <span>{{
            parseTime(scope.row.updateTime, "{y}-{m}-{d} {h}:{i}:{s}")
          }}</span>
        </template>
      </el-table-column>
      <el-table-column
        label="操作"
        align="center"
        class-name="small-padding fixed-width"
      >
        <template #default="scope">
          <el-button
            link
            type="primary"
            icon="Edit"
            @click="handleUpdate(scope.row)"
            v-hasPermi="['setmeal:setmeal:edit']"
            >修改</el-button
          >
          <el-button
            link
            type="primary"
            icon="Delete"
            @click="handleDelete(scope.row)"
            v-hasPermi="['setmeal:setmeal:remove']"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-show="total > 0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

  </div>
</template>

<script setup name="Setmeal">
import { listCategory } from "@/api/category/category";
import { ref } from "vue";
import {
  listSetmeal,
  getSetmeal,
  delSetmeal,
  addSetmeal,
  updateSetmeal,
} from "@/api/setmeal/setmeal";
import { useRouter } from 'vue-router';
import { roleMenuTreeselect, treeselect as menuTreeselect } from "@/api/system/menu";

const { proxy } = getCurrentInstance();
const { seal_system } = proxy.useDict("seal_system");

const setmealList = ref([]);
const setmealDishList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const checkedSetmealDish = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref("");

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    categoryId: null,
    name: null,
    price: null,
    status: null,
  },
  rules: {
    categoryId: [
      { required: true, message: "套餐分类不能为空", trigger: "change" },
    ],
    name: [{ required: true, message: "套餐名称不能为空", trigger: "blur" }],
    price: [{ required: true, message: "套餐价格不能为空", trigger: "blur" }],
    status: [
      { required: true, message: "售卖状态不能为空", trigger: "change" },
    ],
  },
});

const { queryParams, form } = toRefs(data);

//获取分类列表数据
const categoryList = {};

/** 查询菜品分类列表 */
const getCategoryList = async () => {
  try {
    const response = await listCategory(queryParams.value);
    response.rows.forEach((item) => {
      if (item.type === 1) {
        // categoryList.push(item.name);
        categoryList[item.id] = item.name;
      }
    });
  } catch (error) {
    console.log("发生错误：", error);
  }
};

// 定义响应式的分类名称
const categoryNames = ref({});

// 获取分类名称并存储到响应式状态中
const loadCategoryNames = async () => {
  try {
    const response = await listCategory(queryParams.value);
    response.rows.forEach((item) => {
      categoryNames.value[item.id] = item.name;
    });
  } catch (error) {
    console.log("发生错误：", error);
  }
};

// 在组件挂载时调用 loadCategoryNames
onMounted(() => {
  loadCategoryNames();
  getCategoryList();
});

/** 查询套餐列表 */
function getList() {
  loading.value = true;
  listSetmeal(queryParams.value).then((response) => {
    setmealList.value = response.rows;
    total.value = response.total;
    loading.value = false;
  });
}


// 表单重置
function reset() {
  form.value = {
    id: null,
    categoryId: null,
    name: null,
    price: null,
    status: null,
    code: null,
    description: null,
    image: null,
    createTime: null,
    updateTime: null,
  };
  setmealDishList.value = [];
  proxy.resetForm("setmealRef");
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  proxy.resetForm("queryRef");
  handleQuery();
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map((item) => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

const router = useRouter();

const handleAdd = () => {
  router.push('/setmeal/setmeal-info/info');
};
/** 修改按钮操作 */
function handleUpdate(row) {
  const _id = row.id || ids.value;
  if (row && row.id) {
    router.push({ path: `/setmeal/setmeal-info/info/${_id}` });
  } else {
    console.error('无法获取修改项的ID');
  }
}

/** 删除按钮操作 */
function handleDelete(row) {
  const _ids = row.id || ids.value;
  proxy.$modal
    .confirm('是否确认删除套餐编号为"' + _ids + '"的数据项？')
    .then(function () {
      return delSetmeal(_ids);
    })
    .then(() => {
      getList();
      proxy.$modal.msgSuccess("删除成功");
    })
    .catch(() => {});
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download(
    "setmeal/setmeal/export",
    {
      ...queryParams.value,
    },
    `setmeal_${new Date().getTime()}.xlsx`
  );
}

getList();
</script>
