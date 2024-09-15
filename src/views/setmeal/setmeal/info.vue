<template>
  <div class="container">
    <!-- 表单 -->
    <el-form
      ref="setmealRef"
      :model="form"
      :rules="rules"
      :inline="true"
      label-width="180px"
    >
      <el-form-item label="套餐名称:" prop="name">
        <el-input
          v-model="form.name"
          placeholder="请填写套餐名称"
          maxlength="20"
        />
      </el-form-item>
      <el-form-item label="套餐分类:" prop="categoryId">
        <el-select
          v-model="form.categoryId"
          placeholder="请选择套餐分类"
          @change="$forceUpdate()"
          clearable
        >
          <el-option
            v-for="(key, value) in setmealList"
            :key="value"
            :label="key"
            :value="value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="套餐价格:" prop="price">
        <el-input v-model="form.price" placeholder="请设置套餐价格" />
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
      <el-form-item label="套餐图片" prop="image">
        <image-upload v-model="form.image" />
      </el-form-item>
      <el-form-item label="套餐描述:" prop="description">
        <el-input
          v-model="form.description"
          type="textarea"
          :rows="3"
          placeholder="套餐描述，最长200字"
          maxlength="200"
        />
      </el-form-item>

      <el-form-item label="套餐菜品:" class="setmealFood">
        <el-form-item>
          <div class="addDish">
            <el-button
              type="primary"
              v-if="dishTable.length === 0"
              v-hasPermi="['setmeal:setmeal:add']"
              @click="openAddDish"
              >+ 添加菜品</el-button
            >
            <div v-if="dishTable.length !== 0" class="content">
              <div class="action-buttons">
                <el-button type="primary" @click="openAddDish">
                  + 添加菜品
                </el-button>
                <el-form-item
                  class="price-form-item"
                  style="margin-left: auto"
                  label="套餐原价:"
                >
                  {{ totalPrice }}
                </el-form-item>
              </div>
              <div class="table">
                <el-table :data="dishTable" style="width: 100%">
                  <el-table-column
                    prop="name"
                    label="名称"
                    width="180"
                    align="center"
                  ></el-table-column>
                  <el-table-column
                    prop="price"
                    label="原价"
                    width="180"
                    align="center"
                  >
                    <template #default="scope">
                      {{ Number(scope.row.price) }}
                    </template>
                  </el-table-column>
                  <el-table-column prop="copies" label="份数" align="center">
                    <template #default="scope">
                      <el-input-number
                        v-model="scope.row.copies"
                        size="small"
                        :min="1"
                        :max="99"
                        label="描述文字"
                      />
                    </template>
                  </el-table-column>
                  <el-table-column label="操作" width="180px" align="center">
                    <template #default="scope">
                      <el-button
                        type="link"
                        size="small"
                        @click="delDishHandle(scope.$index)"
                      >
                        删除
                      </el-button>
                    </template>
                  </el-table-column>
                </el-table>
              </div>
            </div>
          </div>
        </el-form-item>
      </el-form-item>

      <div class="subBox address">
        <el-form-item>
          <el-button @click="goBack">取消</el-button>
          <el-button type="primary" @click="submitForm(false)">保存</el-button>
          <el-button
            v-if="!form.id"
            type="info"
            class="continue"
            @click="submitForm(true)"
          >
            保存并继续添加套餐
          </el-button>
        </el-form-item>
      </div>
    </el-form>

    <el-dialog
      title="添加菜品"
      class="addDishList"
      v-model="dialogVisible"
      width="60%"
      :before-close="handleClose"
    >
      <el-input
        v-model="queryName"
        class="seachDish"
        placeholder="请输入菜品名称进行搜索"
        style="width: 250px"
        clearable
        @input="getDishForName"
      >
        <template #prefix>
          <el-button
            :icon="Search"
            size="small"
            circle
            @click="getDishForName"
          ></el-button>
        </template>
      </el-input>
      <!-- <AddDish ref="adddish" :check-list="checkList" :seach-key="seachKey" @checkList="getCheckList" /> -->

      <div class="addDishCon">
        <div class="leftCont">
          <div v-show="queryName === ''" class="tabBut">
            <span
              v-for="(item, index) in categoryNameList"
              :key="index"
              :class="{ act: index === keyInd }"
              @click="checkTypeHandle(index, item.id)"
            >
              {{ item.name }}
            </span>
          </div>
          <div class="tabList">
            <div class="table">
              <div v-if="dishAddList.length === 0" style="padding-left: 10px">
                暂无菜品!
              </div>
              <el-checkbox-group
                v-if="dishAddList.length > 0"
                v-model="checkedList"
                @change="checkedListHandle"
              >
                <div
                  v-for="(item, index) in dishAddList"
                  :key="index"
                  class="items"
                >
                  <el-checkbox :label="item">
                    <div class="item">
                      <span style="flex: 3; text-align: left">{{
                        item.name
                      }}</span>
                      <span>{{ item.status === 0 ? "停售" : "在售" }}</span>
                      <span>{{ Number(item.price) }}</span>
                    </div>
                  </el-checkbox>
                </div>
              </el-checkbox-group>
            </div>
          </div>
        </div>
        <div class="ritCont">
          <div class="tit">已选菜品({{ checkedList.length }})</div>
          <div class="items">
            <div v-for="(item, ind) in checkedList" :key="ind" class="item">
              <span>{{ item.name }}</span>
              <span class="price">￥ {{ Number(item.price) }} </span>
              <span class="del" @click="delCheck(ind)">
                <img src="@\assets\images\btn_clean@2x.png" alt="" />
              </span>
            </div>
          </div>
        </div>
      </div>

      <template v-slot:footer>
        <span class="dialog-footer">
          <el-button @click="handleClose">取消</el-button>
          <el-button type="primary" @click="addTableList">确定</el-button>
        </span>
      </template>
    </el-dialog>
  </div>
</template>

<script setup>
import { Search } from "@element-plus/icons-vue";
import { ref, computed, onMounted } from "vue";
import { useRoute } from "vue-router";
import { listCategory } from "@/api/category/category";
import { listDish } from "@/api/dish/dish";
import {
  getSetmeal,
  addSetmeal,
  updateSetmeal,
} from "@/api/setmeal/setmeal";

// 获取当前路由
const route = useRoute();

// 初始化组件数据
onMounted(() => {
  const data = route.params.data;
  if (!data) {
    console.log("这是新增操作");
    // 这里执行新增的逻辑
  } else {
    console.log("这是修改操作，ID为:", data);
    // 根据ID获取数据，执行修改的逻辑
    getSetmeal(data).then((response) => {
      form.value = response.data;
      dishTable.value = response.data.setmealDishList;
    });
  }
});

// 获取字典数据
const { proxy } = getCurrentInstance();
const { seal_system } = proxy.useDict("seal_system");

// 表单数据和规则
const data = reactive({
  form: {},
  rules: {
    name: [{ required: true, message: '请输入套餐名称', trigger: 'blur' }],
    categoryId: [{ required: true, message: '请选择套餐分类', trigger: 'change' }],
    price: [{ required: true, message: '请输入套餐价格', trigger: 'blur' }],
  },
});

const { form, rules } = toRefs(data);
const setmealRef = ref(null);

// 表单重置
function reset() {
  if (setmealRef.value) {
    setmealRef.value.resetFields();
  }
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
  // setmealDishList.value = [];
  dishTable.value = [];
  proxy.resetForm("setmealRef");
}

// Other data
const dialogVisible = ref(false);
const queryName = ref("");
const dishTable = ref([]);
const keyInd = ref(0);
// 获取套餐分类列表数据
const setmealList = {};
// 获取菜品分类列表数据
const categoryNameList = ref([]);
// 搜索和过滤菜品(实现根据输入内容搜索菜品的功能)
const dishAddList = ref([]); // 用于存储当前显示的菜品列表
const checkedList = ref([]); // 已选菜品



//确认选择菜品
const checkedListHandle = () => {
  checkedList.value.forEach((dish) => {
    if (dish.copies === undefined) {
      dish.copies = 1;
    }
  });
};

// 表单提交(实现表单提交逻辑，提交时根据需求保存或继续添加)
const submitForm = async (continueAdding) => {
  try {
    if (!setmealRef.value) {
      console.error("表单引用未找到");
      return;
    }

    await setmealRef.value.validate(async (valid) => {
      if (valid) {
        // 确保 setmealDishList 包含在表单数据中
        form.value.setmealDishList = dishTable.value.map((dish) => ({
          dishId: dish.id,
          name: dish.name,
          price: dish.price,
          copies: dish.copies,
        }));

        let response;
        if (form.value.id) {
          // 修改操作
          response = await updateSetmeal(form.value);
        } else {
          // 新增操作
          response = await addSetmeal(form.value);
        }

        if (response.code === 200) {
          proxy.$message.success("操作成功");
          if (continueAdding) {
            reset(); // 保存并继续添加
          } else {
            goBack(); // 保存后返回
          }
        } else {
          proxy.$message.error(response.msg || "操作失败");
        }
      } else {
        console.log("表单验证失败");
      }
    });
  } catch (error) {
    console.log("提交出错：", error);
  }
};

// 打开添加菜品对话框
function openAddDish() {
  dialogVisible.value = true;
  keyInd.value = 0;
  checkedList.value = []; // 每次打开弹窗时清空 checkedList
  // 初始化菜品数据，加载第一个分类的菜品
  if (categoryNameList.value.length > 0) {
    const firstCategoryId = categoryNameList.value[0].id;
    getDishList(firstCategoryId);
  }
}
// 关闭对话框
const handleClose = () => {
  dialogVisible.value = false;
};

// 获取菜品数据
const getDishForName = async () => {
  try {
    dishAddList.value = [];
    console.log("搜索关键字：", queryName.value);
    const response = await listDish({ name: queryName.value });
    response.rows.forEach((item) => {
      dishAddList.value.push(item);
      console.log("搜索结果：", response);
    });
  } catch (error) {
    console.log("搜索出错：", error);
  }
};

//添加菜品到套餐
const addTableList = () => {
  // 过滤掉已存在的菜品，防止重复添加
  const newDishes = checkedList.value.filter((newDish) => {
    console.log("newDish.id:", newDish.id);
    return !dishTable.value.some(
      (existingDish) => existingDish.id === newDish.id
    );
  });

  // 将过滤后的新菜品添加到 dishTable
  dishTable.value.push(...newDishes);

  // 重新计算总价
  totalPrice.value = totalPrice.value;

  dialogVisible.value = false;
};

// 删除菜品
const delDishHandle = (index) => {
  dishTable.value.splice(index, 1);
  // 重新计算总价
  totalPrice.value = totalPrice.value;
};

// 取消操作
const goBack = () => {};

// 处理选择的菜品
const checkTypeHandle = (index, id) => {
  keyInd.value = index;
  getDishList(id);
};

// 删除已选菜品
const delCheck = (index) => {
  checkedList.value.splice(index, 1);
};

// 获取套餐分类
const getSetmealList = async () => {
  try {
    const response = await listCategory();
    response.rows.forEach((item) => {
      if (item.type === 1) {
        setmealList[item.id] = item.name;
      } else if (item.type === 0) {
        categoryNameList.value.push(item);
      }
    });
  } catch (error) {
    console.log("发生错误：", error);
  }
};

//通过分类ID获取菜品分类包含的菜品列表
const getDishList = async (cid) => {
  try {
    dishAddList.value = [];
    console.log("categoryId:", cid);
    const response = await listDish({ categoryId: cid });
    response.rows.forEach((item) => {
      dishAddList.value.push(item);
      console.log("搜索结果：", response);
    });
  } catch (error) {
    console.log("获取菜品列表出错：", error);
  }
};

// 初始获取分类
getSetmealList();

// 计算总价
const totalPrice = computed(() => {
  return dishTable.value.reduce((total, dish) => {
    // 如果有 copies 属性，计算总价时乘以份数
    const copies = dish.copies ? Number(dish.copies) : 1;
    return total + Number(dish.price) * copies;
  }, 0);
});
</script>

<style>
.container{
  margin: 30px;
}
.container .subBox {
  padding-top: 30px;
  text-align: center;
  border-top: solid 1px #f3f4f7;
}

.addDish .el-input {
  width: 130px;
}
.addDish .el-input-number__increase {
  border-left: solid 1px #ffe1ca;
  background: #fff3ea;
}
.addDish .el-input-number__decrease {
  border-right: solid 1px #ffe1ca;
  background: #fff3ea;
}
.addDish input {
  border: 1px solid #ffe1ca;
}
.addDish .table {
  border: solid 1px #ebeef5;
  border-radius: 3px;
}
.addDish .table th {
  padding: 5px 0;
}
.addDish .table td {
  padding: 7px 0;
}
.addDishList .seachDish {
  position: absolute;
  top: 10px;
  right: 20px;
  box-shadow: var(--el-box-shadow-light);
}
.addDishList .el-dialog__body {
  padding: 0;
  border-bottom: solid 1px #ccc;
}
.addDishList .el-dialog__footer {
  padding-top: 27px;
}

.addDish {
  width: 777px;
}
.addDish .addBut {
  background: #ffc200;
  display: inline-block;
  padding: 0px 20px;
  border-radius: 3px;
  line-height: 40px;
  cursor: pointer;
  border-radius: 4px;
  color: #333333;
  font-weight: 500;
}
.addDish .content {
  background: #fafafb;
  padding: 20px;
  border: solid 1px #ccc;
  border-radius: 3px;
}
.addDishCon {
  padding: 0 20px;
  display: flex;
  line-height: 40px;
}
.addDishCon .leftCont {
  display: flex;
  border-right: solid 2px #e4e7ed;
  width: 60%;
  padding: 15px;
}
.addDishCon .leftCont .tabBut {
  width: 110px;
}
.addDishCon .leftCont .tabBut span {
  display: block;
  text-align: center;
  border-right: solid 2px #f4f4f4;
  cursor: pointer;
}
.addDishCon .leftCont .act {
  border-color: #ffc200 !important;
  color: #ffc200 !important;
}
.addDishCon .leftCont .tabList {
  flex: 1;
  padding: 15px;
}
.addDishCon .leftCont .tabList .table {
  border: solid 1px #f4f4f4;
  border-bottom: solid 1px #f4f4f4;
}
.addDishCon .leftCont .tabList .table .items {
  border-bottom: solid 1px #f4f4f4;
  padding: 0 10px;
  display: flex;
}
.addDishCon .leftCont .tabList .table .items .el-checkbox,
.addDishCon .leftCont .tabList .table .items .el-checkbox__label {
  width: 100%;
}
.addDishCon .leftCont .tabList .table .items .item {
  display: flex;
  padding-right: 20px;
}
.addDishCon .leftCont .tabList .table .items .item span {
  display: inline-block;
  text-align: center;
  flex: 1;
}
.addDishCon .ritCont {
  width: 40%;
  padding: 0 15px;
}
.addDishCon .ritCont .item {
  box-shadow: 0px 1px 4px 3px rgba(0, 0, 0, 0.03);
  display: flex;
  text-align: center;
  padding: 0 10px;
  margin-bottom: 20px;
  border-radius: 6px;
  color: #818693;
}
.addDishCon .ritCont .item span:first-child {
  text-align: left;
  color: #20232a;
}
.addDishCon .ritCont .item .price {
  display: inline-block;
  flex: 1;
}
.addDishCon .ritCont .item .del {
  cursor: pointer;
}
.addDishCon .ritCont .item .del img {
  position: relative;
  top: 5px;
  width: 20px;
}

.addDishCon .el-checkbox__label {
  width: 100%;
}
#combo-add-app .setmealFood .el-form-item__label::before {
  content: "*";
  color: #f56c6c;
  margin-right: 4px;
}

#combo-add-app .uploadImg .el-form-item__label::before {
  content: "*";
  color: #f56c6c;
  margin-right: 4px;
}

.action-buttons {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
}

.price-form-item {
  margin-left: auto;
}
</style>
