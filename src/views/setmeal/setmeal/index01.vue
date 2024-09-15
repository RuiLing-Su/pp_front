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
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['setmeal:setmeal:add']"
          >新增</el-button
        >
        <!-- <el-button type="primary" @click="goToAddPage('add')">新增套餐</el-button> -->

        <el-row style="margin-bottom: 10px">
          <el-col :span="5">
            <el-button
              plain
              type="primary"
              icon="Plus"
              @click="goToAddPage('add')"
            >
              新增
            </el-button>
          </el-col>
        </el-row>
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
      <!-- <el-table-column label="主键" align="center" prop="id" /> -->
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

    <!-- 添加或修改套餐对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="setmealRef" :model="form" :rules="rules" label-width="80px">
        <!-- <el-form-item label="套餐分类" prop="categoryId">
          <el-input v-model="form.categoryId" placeholder="请输入套餐分类" />
        </el-form-item> -->
        <el-form-item label="套餐分类" prop="categoryId">
          <el-select
            v-model="form.categoryId"
            placeholder="请输入套餐分类"
            @change="$forceUpdate()"
            clearable
          >
            <el-option
              v-for="(key, value) in categoryList"
              :key="value"
              :label="key"
              :value="value"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="套餐名称" prop="name">
          <el-input v-model="form.name" placeholder="请输入套餐名称" />
        </el-form-item>
        <el-form-item label="套餐价格" prop="price">
          <el-input v-model="form.price" placeholder="请输入套餐价格" />
        </el-form-item>

        <el-form-item label="售卖状态" prop="status">
          <el-radio-group v-model="form.status">
            <el-radio
              v-for="dict in seal_system"
              :key="dict.value"
              :label="parseInt(dict.value)"
              >{{ dict.label }}</el-radio
            >
          </el-radio-group>
        </el-form-item>
        <el-form-item label="图片" prop="image">
          <image-upload v-model="form.image" />
        </el-form-item>
        <el-form-item label="描述信息" prop="description">
          <el-input
            v-model="form.description"
            type="textarea"
            placeholder="请输入内容"
          />
        </el-form-item>

        <el-form-item label="套餐菜品">
          <el-checkbox
            v-model="menuExpand"
            @change="handleCheckedTreeExpand($event, 'menu')"
            >展开/折叠</el-checkbox
          >
          <el-checkbox
            v-model="menuNodeAll"
            @change="handleCheckedTreeNodeAll($event, 'menu')"
            >全选/全不选</el-checkbox
          >
          <el-checkbox
            v-model="form.menuCheckStrictly"
            @change="handleCheckedTreeConnect($event, 'menu')"
            >父子联动</el-checkbox
          >
          <el-tree
            class="tree-border"
            :data="menuOptions"
            show-checkbox
            ref="menuRef"
            node-key="id"
            :check-strictly="!form.menuCheckStrictly"
            empty-text="加载中，请稍候"
            :props="{ label: 'label', children: 'children' }"
          ></el-tree>
        </el-form-item>

        <el-divider content-position="center">套餐菜品关系信息</el-divider>
        <el-row :gutter="10" class="mb8">
          <el-col :span="1.5">
            <el-button type="primary" icon="Plus" @click="handleAddSetmealDish"
              >添加</el-button
            >
          </el-col>
          <el-col :span="1.5">
            <el-button
              type="danger"
              icon="Delete"
              @click="handleDeleteSetmealDish"
              >删除</el-button
            >
          </el-col>
        </el-row>
        <el-table
          :data="setmealDishList"
          :row-class-name="rowSetmealDishIndex"
          @selection-change="handleSetmealDishSelectionChange"
          ref="setmealDish"
        >
          <el-table-column type="selection" width="50" align="center" />
          <el-table-column label="菜品id" prop="dishId" width="150">
            <template #default="scope">
              <el-input v-model="scope.row.dishId" placeholder="请输入菜品id" />
            </template>
          </el-table-column>
          <el-table-column label="菜品名称 " prop="name" width="150">
            <template #default="scope">
              <el-input
                v-model="scope.row.name"
                placeholder="请输入菜品名称 "
              />
            </template>
          </el-table-column>
          <el-table-column label="菜品原价" prop="price" width="150">
            <template #default="scope">
              <el-input
                v-model="scope.row.price"
                placeholder="请输入菜品原价"
              />
            </template>
          </el-table-column>
          <el-table-column label="份数" prop="copies" width="150">
            <template #default="scope">
              <el-input v-model="scope.row.copies" placeholder="请输入份数" />
            </template>
          </el-table-column>
          <el-table-column label="排序" prop="sort" width="150">
            <template #default="scope">
              <el-input v-model="scope.row.sort" placeholder="请输入排序" />
            </template>
          </el-table-column>
        </el-table>
      </el-form>

      <template #footer>
        <div class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </template>
    </el-dialog>
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
import { useRouter } from "vue-router";
import {
  roleMenuTreeselect,
  treeselect as menuTreeselect,
} from "@/api/system/menu";

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

const { queryParams, form, rules } = toRefs(data);

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

// 取消按钮
function cancel() {
  open.value = false;
  reset();
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

/** 新增按钮操作 */
function handleAdd() {
  reset();
  getMenuTreeselect();
  open.value = true;
  title.value = "添加套餐";
}

const router = useRouter();

const goToAddPage = (data) => {
  router.push("/setmeal/setmeal-info/info/${data}");
};
/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value;
  getSetmeal(_id).then((response) => {
    form.value = response.data;
    setmealDishList.value = response.data.setmealDishList;
    open.value = true;
    title.value = "修改套餐";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["setmealRef"].validate((valid) => {
    if (valid) {
      form.value.setmealDishList = setmealDishList.value;
      if (form.value.id != null) {
        updateSetmeal(form.value).then((response) => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addSetmeal(form.value).then((response) => {
          proxy.$modal.msgSuccess("新增成功");
          open.value = false;
          getList();
        });
      }
    }
  });
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

/** 套餐菜品关系序号 */
function rowSetmealDishIndex({ row, rowIndex }) {
  row.index = rowIndex + 1;
}

/** 套餐菜品关系添加按钮操作 */
function handleAddSetmealDish() {
  let obj = {};
  obj.dishId = "";
  obj.name = "";
  obj.price = "";
  obj.copies = "";
  obj.sort = "";
  setmealDishList.value.push(obj);
}

/** 套餐菜品关系删除按钮操作 */
function handleDeleteSetmealDish() {
  if (checkedSetmealDish.value.length == 0) {
    proxy.$modal.msgError("请先选择要删除的套餐菜品关系数据");
  } else {
    const setmealDishs = setmealDishList.value;
    const checkedSetmealDishs = checkedSetmealDish.value;
    setmealDishList.value = setmealDishs.filter(function (item) {
      return checkedSetmealDishs.indexOf(item.index) == -1;
    });
  }
}

const menuOptions = ref([]);
const menuExpand = ref(false);
const menuNodeAll = ref(false);
const menuRef = ref(null);
const deptRef = ref(null);
/** 查询菜单树结构 */
function getMenuTreeselect() {
  menuTreeselect().then((response) => {
    menuOptions.value = response.data;
  });
}
/** 根据角色ID查询菜单树结构 */
function getRoleMenuTreeselect(roleId) {
  return roleMenuTreeselect(roleId).then((response) => {
    menuOptions.value = response.menus;
    return response;
  });
}
/** 树权限（展开/折叠）*/
function handleCheckedTreeExpand(value, type) {
  if (type == "menu") {
    let treeList = menuOptions.value;
    for (let i = 0; i < treeList.length; i++) {
      menuRef.value.store.nodesMap[treeList[i].id].expanded = value;
    }
  } else if (type == "dept") {
    let treeList = deptOptions.value;
    for (let i = 0; i < treeList.length; i++) {
      deptRef.value.store.nodesMap[treeList[i].id].expanded = value;
    }
  }
}
/** 树权限（全选/全不选） */
function handleCheckedTreeNodeAll(value, type) {
  if (type == "menu") {
    menuRef.value.setCheckedNodes(value ? menuOptions.value : []);
  } else if (type == "dept") {
    deptRef.value.setCheckedNodes(value ? deptOptions.value : []);
  }
}

/** 树权限（父子联动） */
function handleCheckedTreeConnect(value, type) {
  if (type == "menu") {
    form.value.menuCheckStrictly = value ? true : false;
  } else if (type == "dept") {
    form.value.deptCheckStrictly = value ? true : false;
  }
}
/** 所有菜单节点数据 */
function getMenuAllCheckedKeys() {
  // 目前被选中的菜单节点
  let checkedKeys = menuRef.value.getCheckedKeys();
  // 半选中的菜单节点
  let halfCheckedKeys = menuRef.value.getHalfCheckedKeys();
  checkedKeys.unshift.apply(checkedKeys, halfCheckedKeys);
  return checkedKeys;
}

// 打开对话框
const openDialog = () => {
  dialogVisible.value = true;
};

// 关闭对话框
const handleClose = () => {
  dialogVisible.value = false;
};

/** 复选框选中数据 */
function handleSetmealDishSelectionChange(selection) {
  checkedSetmealDish.value = selection.map((item) => item.index);
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
