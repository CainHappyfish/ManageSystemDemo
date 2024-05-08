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
// 编辑表单
const editForm = reactive({
  user: '',
  status: [],
  time: [],
})

// 日期周期
const pickerOptions = reactive({
  shortcuts: [
          {
            text: "最近一周",
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
const editFormVisible = ref(false)    // 编辑弹框

// 表格数据
const tableData = ref([])
// 本地存储
const list = ref([])
// 表单填写
const form = reactive({
  user: '',
  status: [],
  time: [],
})

function addnew() {

}


const onSubmit = () => {
  console.log('submit!')
}
</script>

<template>

  <div id="app">
    <el-card style="max-width: 600px">
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

          <!-- 增加新项目 -->
          <div class="new">
            <el-button type="primary" round size="large" @click="dialogFormVisible = true">添加新状态</el-button>
            <el-dialog
              v-model="dialogFormVisible"
              title="新增订单"
              width="500"
              :before-close="handleClose"
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
                    value-format="yyyy-MM-dd HH:mm:ss"
                    placeholder="选择日期时间"
                  ></el-date-picker>

                </el-form-item>



              </el-form>

              <template #footer>
                <div class="dialog-footer">
                  <el-button @click="dialogFormVisible = false">Cancel</el-button>
                  <el-button type="primary" @click="dialogFormVisible = false">
                    Confirm
                  </el-button>
                </div>
              </template>
            </el-dialog>

          </div>





        </div>
      </template>


      <el-table :data="tableData" height="250" border style="width: 100%">
          <el-table-column prop="time" label="日期" width="180"></el-table-column>
          <el-table-column prop="user" label="收货人" width="180"></el-table-column>
          <el-table-column prop="status" label="订单状态"></el-table-column>
      </el-table>


      <template #footer>Footer content</template>
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