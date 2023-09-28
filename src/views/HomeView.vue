<template>
  <div class="container">
    <el-form ref="ruleFormRef" :model="ruleForm" :rules="formRules">
      <el-row :gutter="10">
        <el-col :span="20">
          <el-form-item prop="name">
            <el-input v-model="ruleForm.name"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="4">
          <el-form-item>
            <el-button @click.stop="submitForm(ruleFormRef)">Add</el-button>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <el-table
      :data="list"
      :border="true"
      :stripe="true"
      :show-header="false"
      @selection-change="onChanged"
      size="small"
    >
      <el-table-column>
        <template #default="scope">
          <el-checkbox
            :checked="scope.row.complete"
            @change="onChanged(scope)"
          ></el-checkbox>
        </template>
      </el-table-column>
      <el-table-column prop="name" />
      <el-table-column>
        <template #default="scope">
          <el-tag type="success" v-if="scope.row.complete">完成</el-tag>
          <el-tag v-else>待定</el-tag>
        </template>
      </el-table-column>
      <el-table-column>
        <template #default="scope">
          <el-button
            type="danger"
            size="small"
            @click.stop="() => deleteItem(scope.row._id)"
            >删除</el-button
          >
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, reactive } from "vue";
import { Cloud } from "laf-client-sdk";
import { ElMessageBox, ElMessage } from "element-plus";
import type { FormInstance, FormRules } from "element-plus";

interface TodoItem {
  _id?: string;
  name: string;
  complete: boolean;
}
const ruleFormRef = ref<FormInstance>();
const list = ref<TodoItem[]>([]);
const ruleForm = reactive<TodoItem>({
  name: "",
  complete: false,
});
const formRules = reactive<FormRules<TodoItem>>({
  name: [
    {
      required: true,
      message: "必填",
      trigger: "blur",
    },
  ],
});
// 创建 cloud 对象
const cloud = new Cloud({
  baseUrl: "https://jac6h5.laf.run",
  getAccessToken: () => "", // 这里先为空
});

async function getList() {
  // 调用 get-list 云函数不传参
  const res = await cloud.invoke("get-list");
  list.value = res.data;
}

async function submitForm(formEl: FormInstance) {
  if (!formEl) return;
  await formEl.validate(async (valid, fields) => {
    if (valid) {
      // 调用 add-todo 传入参数 obj
      await cloud.invoke("add-todo", ruleForm);
      ElMessage({
        type: "success",
        message: "添加成功",
      });
      ruleForm.name = "";
      getList();
    } else {
      console.log("error submit!", fields);
    }
  });
}

async function complete(index: number, id: string) {
  list.value[index].complete = !list.value[index].complete;
  // 调用 update-todo 传入参数
  await cloud.invoke("update-todo", {
    id,
    data: list.value[index],
  });
}

async function deleteItem(id: string) {
  ElMessageBox.confirm("确定要删除吗", "警告", {
    confirmButtonText: "确定",
    cancelButtonText: "取消",
  })
    .then(async (action) => {
      if (action === "confirm") {
        // 调用 del-todo 传入参数
        await cloud.invoke("del-todo", { id });
        getList();
        ElMessage({
          type: "success",
          message: "删除成功",
        });
      }
    })
    .catch(() => {});
}

function onChanged(scope: any) {
  const { row, $index } = scope;
  complete($index, row._id);
}

onMounted(() => {
  getList();
});
</script>

<style lang="css" scoped>
.container {
  width: 100%;
  height: 100%;
  padding: 20px;
  box-sizing: border-box;
}
</style>
