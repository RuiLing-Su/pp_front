<template>
  <div class="app-container">
    <el-form :model="queryParams" ref="queryRef" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="订单号" prop="number">
        <el-input
          v-model="queryParams.number"
          placeholder="请输入订单号"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item label="订单状态" prop="status">
        <el-select v-model="queryParams.status" placeholder="请选择订单状态" clearable>
          <el-option
            v-for="dict in order_system"
            :key="dict.value"
            :label="dict.label"
            :value="dict.value"
          />
        </el-select>
      </el-form-item>
      <el-form-item label="下单时间" style="width: 308px">
        <el-date-picker
          v-model="daterangeOrderTime"
          value-format="YYYY-MM-DD"
          type="daterange"
          range-separator="-"
          start-placeholder="开始日期"
          end-placeholder="结束日期"
        ></el-date-picker>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="Search" @click="handleQuery">搜索</el-button>
        <el-button icon="Refresh" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <el-row :gutter="10" class="mb8">
<!--       
      <el-col :span="1.5">
        <el-button
          type="primary"
          plain
          icon="Plus"
          @click="handleAdd"
          v-hasPermi="['orders:orders:add']"
        >新增</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="success"
          plain
          icon="Edit"
          :disabled="single"
          @click="handleUpdate"
          v-hasPermi="['orders:orders:edit']"
        >修改</el-button>
      </el-col>
      <el-col :span="1.5">
        <el-button
          type="danger"
          plain
          icon="Delete"
          :disabled="multiple"
          @click="handleDelete"
          v-hasPermi="['orders:orders:remove']"
        >删除</el-button>
      </el-col> -->

      <el-col :span="1.5">
        <el-button
          type="warning"
          plain
          icon="Download"
          @click="handleExport"
          v-hasPermi="['orders:orders:export']"
        >导出</el-button>
      </el-col>
      <right-toolbar v-model:showSearch="showSearch" @queryTable="getList"></right-toolbar>
    </el-row>

    <el-table v-loading="loading" :data="ordersList" @selection-change="handleSelectionChange">
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="订单号" align="center" prop="number" />
      <el-table-column label="订单状态" align="center" prop="status">
        <template #default="scope">
          <dict-tag :options="order_system" :value="scope.row.status"/>
        </template>
      </el-table-column>
      <el-table-column label="用户手机号" align="center" prop="phone" />
      <el-table-column label="餐桌号" align="center" prop="tableNo" />
      <!-- <el-table-column label="主键" align="center" prop="id" /> -->
      <el-table-column label="下单时间" align="center" prop="checkoutTime" width="180">
        <template #default="scope">
          <span>{{ parseTime(scope.row.checkoutTime, '{y}-{m}-{d} {h}:{i}:{s}') }}</span>
        </template>
      </el-table-column>
      <el-table-column label="实收金额" align="center">
        <template #default="scope">
          <span>￥{{ scope.row.amount }}</span>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template #default="scope">
          <!-- <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['orders:orders:edit']">修改</el-button> -->
          <el-button link type="primary" icon="Edit" @click="handleUpdate(scope.row)" v-hasPermi="['orders:orders:edit']">查看</el-button>
          <!-- <el-button link type="primary" icon="Delete" @click="handleDelete(scope.row)" v-hasPermi="['orders:orders:remove']">删除</el-button> -->
           <el-button v-if="scope.row.status == 2" link type="primary" @click="cancelOrDeliveryOrComplete(3, scope.row.number)" v-hasPermi="['orders:orders:edit']">送出</el-button>
           <el-button v-if="scope.row.status == 3" link type="primary" @click="cancelOrDeliveryOrComplete(4, scope.row.number)" v-hasPermi="['orders:orders:edit']">完成</el-button>
        </template>
      </el-table-column>
    </el-table>
    
    <pagination
      v-show="total>0"
      :total="total"
      v-model:page="queryParams.pageNum"
      v-model:limit="queryParams.pageSize"
      @pagination="getList"
    />

    <!-- 添加或修改订单对话框 -->
    <el-dialog :title="title" v-model="open" width="500px" append-to-body>
      <el-form ref="ordersRef" :model="form" :rules="rules" label-width="80px">
        <el-divider content-position="center">订单明细信息</el-divider>
        <el-row :gutter="10" class="mb8">
          <el-col :span="1.5">
            <el-button type="primary" icon="Plus" @click="handleAddOrderDetail">添加</el-button>
          </el-col>
          <el-col :span="1.5">
            <el-button type="danger" icon="Delete" @click="handleDeleteOrderDetail">删除</el-button>
          </el-col>
        </el-row>
        <el-table :data="orderDetailList" :row-class-name="rowOrderDetailIndex" @selection-change="handleOrderDetailSelectionChange" ref="orderDetail">
          <el-table-column type="selection" width="50" align="center" />
          <el-table-column label="序号" align="center" prop="index" width="50"/>
          <el-table-column label="名字" prop="name" width="150">
            <template #default="scope">
              <el-input v-model="scope.row.name" placeholder="请输入名字" />
            </template>
          </el-table-column>
          <el-table-column label="菜品id" prop="dishId" width="150">
            <template #default="scope">
              <el-input v-model="scope.row.dishId" placeholder="请输入菜品id" />
            </template>
          </el-table-column>
          <el-table-column label="套餐id" prop="setmealId" width="150">
            <template #default="scope">
              <el-input v-model="scope.row.setmealId" placeholder="请输入套餐id" />
            </template>
          </el-table-column>
          <el-table-column label="口味" prop="dishFlavor" width="150">
            <template #default="scope">
              <el-input v-model="scope.row.dishFlavor" placeholder="请输入口味" />
            </template>
          </el-table-column>
          <el-table-column label="数量" prop="number" width="150">
            <template #default="scope">
              <el-input v-model="scope.row.number" placeholder="请输入数量" />
            </template>
          </el-table-column>
          <el-table-column label="金额" prop="amount" width="150">
            <template #default="scope">
              <el-input v-model="scope.row.amount" placeholder="请输入金额" />
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

<script setup name="Orders">
import { listOrders, getOrders, delOrders, addOrders, updateOrders } from "@/api/orders/orders";

const { proxy } = getCurrentInstance();
const { order_system } = proxy.useDict('order_system');

const ordersList = ref([]);
const orderDetailList = ref([]);
const open = ref(false);
const loading = ref(true);
const showSearch = ref(true);
const ids = ref([]);
const checkedOrderDetail = ref([]);
const single = ref(true);
const multiple = ref(true);
const total = ref(0);
const title = ref("");
const daterangeOrderTime = ref([]);

const data = reactive({
  form: {},
  queryParams: {
    pageNum: 1,
    pageSize: 10,
    number: null,
    status: null,
    orderTime: null,
  },
  rules: {
  }
});

const { queryParams, form, rules } = toRefs(data);

/** 查询订单列表 */
function getList() {
  loading.value = true;
  queryParams.value.params = {};
  if (null != daterangeOrderTime && '' != daterangeOrderTime) {
    queryParams.value.params["beginOrderTime"] = daterangeOrderTime.value[0];
    queryParams.value.params["endOrderTime"] = daterangeOrderTime.value[1];
  }
  listOrders(queryParams.value).then(response => {
    ordersList.value = response.rows;
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
    tableNo: null,
    id: null,
    number: null,
    status: null,
    userId: null,
    orderTime: null,
    checkoutTime: null,
    payMethod: null,
    amount: null,
    remark: null,
    phone: null
  };
  orderDetailList.value = [];
  proxy.resetForm("ordersRef");
}

/** 搜索按钮操作 */
function handleQuery() {
  queryParams.value.pageNum = 1;
  getList();
}

/** 重置按钮操作 */
function resetQuery() {
  daterangeOrderTime.value = [];
  proxy.resetForm("queryRef");
  handleQuery();
}

// 多选框选中数据
function handleSelectionChange(selection) {
  ids.value = selection.map(item => item.id);
  single.value = selection.length != 1;
  multiple.value = !selection.length;
}

/** 新增按钮操作 */
function handleAdd() {
  reset();
  open.value = true;
  title.value = "添加订单";
}

/** 修改按钮操作 */
function handleUpdate(row) {
  reset();
  const _id = row.id || ids.value
  getOrders(_id).then(response => {
    form.value = response.data;
    orderDetailList.value = response.data.orderDetailList;
    open.value = true;
    title.value = "修改订单";
  });
}

/** 提交按钮 */
function submitForm() {
  proxy.$refs["ordersRef"].validate(valid => {
    if (valid) {
      form.value.orderDetailList = orderDetailList.value;
      if (form.value.id != null) {
        updateOrders(form.value).then(response => {
          proxy.$modal.msgSuccess("修改成功");
          open.value = false;
          getList();
        });
      } else {
        addOrders(form.value).then(response => {
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
  proxy.$modal.confirm('是否确认删除订单编号为"' + _ids + '"的数据项？').then(function() {
    return delOrders(_ids);
  }).then(() => {
    getList();
    proxy.$modal.msgSuccess("删除成功");
  }).catch(() => {});
}

/** 订单明细序号 */
function rowOrderDetailIndex({ row, rowIndex }) {
  row.index = rowIndex + 1;
}

/** 订单明细添加按钮操作 */
function handleAddOrderDetail() {
  let obj = {};
  obj.name = "";
  obj.image = "";
  obj.dishId = "";
  obj.setmealId = "";
  obj.dishFlavor = "";
  obj.number = "";
  obj.amount = "";
  orderDetailList.value.push(obj);
}

/** 订单明细删除按钮操作 */
function handleDeleteOrderDetail() {
  if (checkedOrderDetail.value.length == 0) {
    proxy.$modal.msgError("请先选择要删除的订单明细数据");
  } else {
    const orderDetails = orderDetailList.value;
    const checkedOrderDetails = checkedOrderDetail.value;
    orderDetailList.value = orderDetails.filter(function(item) {
      return checkedOrderDetails.indexOf(item.index) == -1
    });
  }
}

/** 复选框选中数据 */
function handleOrderDetailSelectionChange(selection) {
  checkedOrderDetail.value = selection.map(item => item.index)
}

/** 导出按钮操作 */
function handleExport() {
  proxy.download('orders/orders/export', {
    ...queryParams.value
  }, `orders_${new Date().getTime()}.xlsx`)
}

/** 修改按钮操作 */
// function handleUpdate(row) {
//   reset();
//   const _id = row.id || ids.value
//   getOrders(_id).then(response => {
//     form.value = response.data;
//     orderDetailList.value = response.data.orderDetailList;
//     open.value = true;
//     title.value = "修改订单";
//   });
// }

// function submitForm() {
//   proxy.$refs["ordersRef"].validate(valid => {
//     if (valid) {
//       form.value.orderDetailList = orderDetailList.value;
//       if (form.value.id != null) {
//         updateOrders(form.value).then(response => {
//           proxy.$modal.msgSuccess("修改成功");
//           open.value = false;
//           getList();
//         });
//       } 
// 取消订单、派送订单、完成订单
function cancelOrDeliveryOrComplete(status, id){
  reset();
  const data = {
    id: id,
    status: status
  }
  updateOrders(data).then(response => {
    if (response && response.code == 200){
      proxy.$modal.msgSuccess("修改成功");
      getList();
    }else{
      proxy.$modal.msgError(response.message || "修改失败");
    }
  }).catch((error) => {
    proxy.$modal.msgError(error.message || "修改失败");
  });
}
getList();
</script>
