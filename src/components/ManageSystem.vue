<script setup lang="ts">
import { reactive, ref } from 'vue'


// 表单数据
// 初始查询表单
const formInline = reactive({
  user: '',
  status: [],
  time: [],
})
// 查询表单
const searchForm = reactive({
  user: '',
  status: [],
  time: [],
})


// 日期周期
const pickerOptions = reactive({
  shortcuts: [
          {
            text: "最近一周",
            // ts.json
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近一个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit("pick", [start, end]);
            }
          },
          {
            text: "最近三个月",
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit("pick", [start, end]);
            }
          }
        ]
})

const dialogFormVisible = ref(false)  // 新增弹框


// 表格数据
const tableData = ref([])
// 本地存储
const list = ref([])
// 表单填写
const form = reactive({
  user: '',
  status: [] as string[],
  time: [],
})

// 本地存储函数
function addNew() {

  // 将表单数据添加到列表数组的开头
  list.value.unshift(form);
  console.log(list)
  list.value.sort((a, b) => {
    return new Date(b.time).getTime() - new Date(a.time).getTime();
  }); //降序

  window.localStorage.setItem("list", JSON.stringify(list.value));
   // 清空表单数据
  form.user = '';
  form.status = [];
  form.time = [];
  dialogFormVisible.value = false;//隐藏新增弹框
  showList();//数据渲染
}

// 数据渲染
function showList() {
  const storedList = JSON.parse(window.localStorage.getItem("list") || '[]') || [];
  tableData.value = storedList.map((item) => {
    item.time = new Date(item.time).toLocaleString();
    return item;
  });
}

const editFormVisible = ref(false)    // 编辑弹框
// 编辑每项内容
const rowItem = reactive({
  user: '',
  status: [] as string[],
  time: [],
})
// 编辑表单
const editForm = reactive({
  user: '',
  status: [] as string[],
  time: [],
})

// 编辑按钮事件
function handleEdit(row) {
  editFormVisible.value = true
  rowItem.value = row
  editForm.user = rowItem.user
  editForm.status = rowItem.status
  editForm.time = rowItem.time
}

// 删除按钮事件
function handleDelete(row: any) {

}



const onSubmit = () => {
  console.log('submit!')
}
</script>

<template>

  <div id="app">
    <el-card style="max-width: 700px">
      <template #header>
        <div slot="header" class="card-header">
          <!-- 收货人 -->
          <el-form size="small" :inline="true" :model="formInline" class="demo-form-inline" label-position="top">
            <el-form-item label="收货人">
              <el-input v-model="formInline.user" placeholder="收货人" size="large"></el-input>
            </el-form-item>

          <!-- 订单状态 -->
          <el-form-item label="订单状态">
            <el-select v-model="formInline.status" placeholder="订单状态" style="width: 200px" size="large">
              <el-option label="未受理" value="未受理"></el-option>
              <el-option label="已受理" value="已受理"></el-option>
              <el-option label="已送达" value="已送达"></el-option>
            </el-select>
          </el-form-item>

          <!-- 时间选择 -->
          <el-form-item>
            <div class="date-picker">
              <el-date-picker
              v-model="formInline.time"
              type="datetimerange"
              :picker-options="pickerOptions"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              size="large"
              ></el-date-picker>
            </div>
          </el-form-item>


          <el-form-item>
              <el-button type="primary" round size="large" @click="onSubmit">查询</el-button>
            </el-form-item>

          </el-form>


        </div>
      </template>


      <el-table :data="tableData" height="250" style="width: 100%; margin-top: 0">
        <el-table-column prop="time" label="日期" width="180"></el-table-column>
        <el-table-column prop="user" label="收货人" width="180"></el-table-column>
        <el-table-column prop="status" label="订单状态"></el-table-column>
        <el-table-column label="操作" style="text-align: center" >
          <template #default="scope">
            <el-button size="small" @click="handleEdit(scope.row)">
              编辑
            </el-button>
            <el-dialog
                v-model="editFormVisible"
                title="编辑订单"
            >
              <el-form size="default" style="width: 500px" label-width="100px" :model="editForm">
                <el-form-item label="收货人">
                    <el-input style="width:300px;" v-model="editForm.user" placeholder="收货人"></el-input>
                  </el-form-item>
                  <!-- 订单状态 -->
                  <el-form-item label="订单状态">
                    <el-select style="width:300px;" v-model="editForm.status" placeholder="订单状态">
                      <el-option label="未发货" value="未受理"></el-option>
                      <el-option label="已受理" value="已受理"></el-option>
                      <el-option label="已送达" value="已送达"></el-option>
                    </el-select>
                  </el-form-item>
                <!-- 下单时间 -->
                  <el-form-item label="下单时间">
                    <el-date-picker
                      style="width:300px"
                      v-model="editForm.time"
                      value-format="yyyy-MM-dd HH:mm:ss"
                      type="datetime"
                      placeholder="选择日期时间"
                    ></el-date-picker>
                  </el-form-item>
              </el-form>
              <template #footer>

              </template>
            </el-dialog>
            <el-button size="small" type="danger" @click="handleDelete(scope.row)" style="margin-left: 10px">
              删除
            </el-button>
          </template>

        </el-table-column>

      </el-table>


      <template #footer>
        <!-- 增加新项目 -->
          <div class="new">
            <el-button type="primary" round size="large" @click="dialogFormVisible = true">添加新状态</el-button>
            <el-dialog
              v-model="dialogFormVisible"
              title="新增订单"
              width="500"
            >
            <!-- 对话框表单 -->
              <el-form size="small" style="width:450px" label-width="100px" :model="form">
                <!-- 收货人 -->
                <el-form-item label="收货人">
                  <el-input
                    style="width:280px;margin-left:-80px"
                    v-model="form.user"
                    placeholder="收货人"
                  ></el-input>
                </el-form-item>
                <!-- 状态 -->
                <el-form-item label="订单状态">
                  <el-select
                    v-model="form.status"
                    style="width:280px;margin-left:-80px"
                    placeholder="订单状态">
                    <el-option label="未发货" value="未受理"></el-option>
                    <el-option label="已受理" value="已受理"></el-option>
                    <el-option label="已送达" value="已送达"></el-option>
                  </el-select>
                </el-form-item>
                <!-- 时间选择 -->
                <el-form-item>
                  <el-date-picker
                    style="width:280px;margin-left:-80px"
                    v-model="form.time"
                    type="datetime"
                    placeholder="选择日期时间"
                  ></el-date-picker>

                </el-form-item>



              </el-form>

              <template #footer>
                <div class="dialog-footer">
                  <el-button @click="dialogFormVisible = false">Cancel</el-button>
                  <el-button type="primary" @click="addNew">
                    Confirm
                  </el-button>
                </div>
              </template>
            </el-dialog>

          </div>
      </template>
    </el-card>

  </div>
</template>

<style scoped>
#app {
  display: flex;
  justify-content: center;
  place-items: center;
  align-items: center;
}
</style>