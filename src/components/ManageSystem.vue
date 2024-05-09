<script setup lang="ts">
import {reactive, ref} from 'vue'

interface ItemMessage {
  user: string
  status: string
  time: string
}


const dialogFormVisible = ref(false)  // 新增弹框


// 表格数据
const tableData = ref<ItemMessage[]>([])
// 本地存储
let list = ref<ItemMessage[]>([])
// 表单填写
const form = reactive({
  user: '',
  status: '',
  time: '',
})

// 本地存储函数
function addNew() {
  const timeValue = form.time ? form.time : '未填写';
  const temp = {
    user: form.user,
    status: form.status,
    time: timeValue
  }

  // 将表单数据添加到列表数组的开头
  list.value.unshift(temp);
  list.value.sort((a, b) => {
    return new Date(b.time).getTime() - new Date(a.time).getTime();
  }); //降序


  window.localStorage.setItem("list", JSON.stringify(list.value));
   // 清空表单数据

  form.user = '';
  form.status = '';
  form.time = '';

  console.log("list2",list.value[0])
  dialogFormVisible.value = false;//隐藏新增弹框

  showList()//数据渲染
}

// 数据渲染
function showList() {
  const storedList = JSON.parse(window.localStorage.getItem("list") || '[]') ;
  console.log(storedList)
  tableData.value = storedList.map((item : any) => {
    if (item.time && !isNaN(Date.parse(item.time))) {
      // 如果是有效的日期字符串，转换为本地时间字符串
      item.time = new Date(item.time).toLocaleString();
    } else {
      // 如果不是有效的日期字符串，设置为 '无时间'
      item.time = '未填写';
    }
    return item as ItemMessage;
  });
}

const editFormVisible = ref(false)    // 编辑弹框

interface RowData {
  user: string;
  status: string;
  time: string; // 假设时间是以字符串形式存储的日期
}

// 编辑每项内容
const rowItem = reactive({
  user: '',
  status: '',
  time: '',
})

// 编辑表单
const editForm = reactive({
  user: '',
  status: '',
  time: '',
})

// 编辑按钮事件
function handleEdit(row : RowData) {
  editFormVisible.value = true
  rowItem.user = row.user;
  rowItem.status = row.status;
  rowItem.time = row.time;
  editForm.user = rowItem.user
  editForm.status = rowItem.status
  editForm.time = rowItem.time

}
import { ElMessageBox, ElMessage } from 'element-plus';
// 提交修改
function submitEdit() {
  // 找到要更新的行的索引
  const index = list.value.findIndex((item) => item.user === rowItem.user)
  if (index !== -1) {
    // 更新数据
    list.value[index] = { ...editForm, time: new Date(editForm.time).toISOString() };
    list.value.sort((a, b) => new Date(b.time).getTime() - new Date(a.time).getTime());
    window.localStorage.setItem("list", JSON.stringify(list.value));
    showList();
    ElMessage({
      type: 'success',
      message: '已编辑'
    })

  }else {
    alert("Wrong!")
  }

  editFormVisible.value = false
}
// 删除按钮事件
function handleDelete(row: ItemMessage) {
  ElMessageBox.confirm(
      '确定删除该记录？',
      '警告',
      {
        confirmButtonText: 'Confirm',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }
  )
      .then(() => {
        // 找到要更新的行的索引
        const index = list.value.indexOf(row)
        list.value.splice(index, 1)
        window.localStorage.setItem("list", JSON.stringify(list.value))
        showList()                   // 更新表格数据
        ElMessage({
          type: 'success',
          message: '已删除'
        })
      })
      .catch(() => {
        ElMessage({
          type: 'info',
          message: '已取消删除'
        })
      })
}

// 初始查询表单
const formInline = reactive({
  user: '',
  status: '',
  time: '',
})
// 查询数据
const searchData = ref<ItemMessage[]>([])
// 查询表单
const searchForm = reactive({
  user: '',
  status: '',
  time: '',
})
// 处理查询请求
function onSubmit() {
  searchForm.user = formInline.user
  searchForm.status = formInline.status
  searchForm.time = formInline.time
  searchData.value = []
}


// 页码实现

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
                range-separator="至"
                start-placeholder="开始日期"
                end-placeholder="结束日期"
                size="large"
                ></el-date-picker>
              </div>
            </el-form-item>

            <!--  查询 -->
            <el-form-item>
                <el-button type="primary" round size="large" @click="onSubmit">查询</el-button>
            </el-form-item>

          </el-form>


        </div>
      </template>


      <el-table :data="tableData" height="250" style="width: 100%; margin-top: 0">
        <el-table-column prop="time" label="时间" width="180"></el-table-column>
        <el-table-column prop="user" label="收货人" width="100"></el-table-column>
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
                      <el-option label="未受理" value="未受理"></el-option>
                      <el-option label="已受理" value="已受理"></el-option>
                      <el-option label="已送达" value="已送达"></el-option>
                    </el-select>
                  </el-form-item>
                <!-- 下单时间 -->
                  <el-form-item label="下单时间">
                    <el-date-picker
                      style="width:300px"
                      v-model="editForm.time"
                      type="datetime"
                      placeholder="选择日期时间"
                    ></el-date-picker>
                  </el-form-item>
              </el-form>
              <template #footer>
                <div class="dialog-footer">
                  <el-button @click="editFormVisible = false">Cancel</el-button>
                  <el-button type="primary" @click="submitEdit">
                    Confirm
                  </el-button>
                </div>

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
              width="350px"
            >
            <!-- 对话框表单 -->
              <el-form size="large" style="width:450px" label-width="100px" :model="form">
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
                    <el-option label="未受理" value="未受理"></el-option>
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
.new {
  display: flex; /* 使用 Flexbox */
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
}
.dialog-footer {
  display: flex; /* 使用 Flexbox */
  justify-content: center; /* 水平居中 */
  align-items: center; /* 垂直居中 */
  /* 如果需要垂直居中，可以添加以下属性 */
  height: 60px; /* 对话框底部的高度，根据实际情况调整 */
}
</style>