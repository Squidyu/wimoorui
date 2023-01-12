 

<template>
  <div class="app-container">
    <!-- 搜索表单 -->
    <el-form ref="queryFormRef" :model="state.queryParams" :inline="true">
      <el-form-item>
        <el-button type="success" :icon="Plus" @click="handleAdd"
          >新增</el-button
        >
        <el-button
          type="danger"
          :icon="Delete"
          :disabled="state.multiple"
          @click="handleDelete"
          >删除
        </el-button>
      </el-form-item>

      <el-form-item prop="name">
        <el-input
          v-model="state.queryParams.name"
          placeholder="字典名称"
          clearable
          @keyup.enter="handleQuery"
        />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" :icon="Search" @click="handleQuery()"
          >搜索</el-button
        >
        <el-button :icon="Refresh" @click="resetQuery()">重置</el-button>
      </el-form-item>
    </el-form>

    <!-- 数据表格 -->
    <el-table
      highlight-current-row
      :data="dictList"
      v-loading="loading"
      @row-click="handleRowClick"
      @selection-change="handleSelectionChange"
      border
    >
      <el-table-column type="selection" width="55" align="center" />
      <el-table-column label="字典名称" prop="name" width="120" />
      <el-table-column label="字典编码" prop="code" />
      <el-table-column label="状态" align="center" width="80">
        <template #default="scope">
          <el-tag v-if="scope.row.status === 1" type="success">启用</el-tag>
          <el-tag v-else type="info">禁用</el-tag>
        </template>
      </el-table-column>

      <el-table-column label="操作" align="center" width="150">
        <template #default="scope">
          <el-button
            type="primary"
             link
            @click.stop="handleUpdate(scope.row)"
           >编辑</el-button>
          <el-button
            type="danger"
            link
            @click.stop="handleDelete(scope.row)"
          >删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <pagination
      v-if="total > 0"
      :total="total"
      v-model:page="queryParams.page"
      v-model:limit="queryParams.limit"
      @pagination="handleQuery"
    />

    <!-- 弹窗表单 -->
    <el-dialog
      :title="dialog.title"
      v-model="dialog.visible"
      width="500px"
      @close="cancel"
    >
      <el-form
        ref="dataFormRef"
        :model="formData"
        :rules="rules"
        label-width="80px"
      >
        <el-form-item label="字典名称" prop="name">
          <el-input v-model="formData.name" placeholder="请输入字典名称" />
        </el-form-item>
        <el-form-item label="字典编码" prop="code">
          <el-input v-model="formData.code" placeholder="请输入字典编码" />
        </el-form-item>
        <el-form-item label="状态" prop="status">
          <el-radio-group v-model="formData.status">
            <el-radio :label="1">正常</el-radio>
            <el-radio :label="0">停用</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="备注" prop="remark">
          <el-input
            v-model="formData.remark"
            type="textarea"
            placeholder="请输入内容"
            :autosize="{ minRows: 2, maxRows: 4 }"
          />
        </el-form-item>
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

<script  >
import { onMounted, reactive, ref, toRefs } from 'vue';
 
import { Search, Plus, Edit, Refresh, Delete } from '@element-plus/icons-vue';
import { ElForm, ElMessage, ElMessageBox } from 'element-plus';
 import {
   updateDictType,
   listPageDictTypes, 
   getDictFormData,
   deleteDictTypes,
   addDictType,
 } from '@/api/sys/admin/dict';
export default {
  name: 'dictType',
  components:{Search,Plus,Edit,Refresh,Delete},
  emits:["dictClick"],
  setup(props,context){
			const queryFormRef = ref(ElForm);
			const dataFormRef = ref(ElForm);
			const state = reactive({
			  loading: true,
			  // 选中ID数组
			  ids: [],
			  // 非单个禁用
			  single: true,
			  // 非多个禁用
			  multiple: true,
			  queryParams: {
				page: 1,
				limit: 10,
				name:'',
			  } ,
			  dictList: [],
			  total: 0,
			  dialog: { visible: false } ,
			  formData: {
				status: 1,
			  } ,
			  rules: {
				name: [{ required: true, message: '请输入字典名称', trigger: 'blur' }],
				code: [{ required: true, message: '请输入字典编码', trigger: 'blur' }],
			  },
			});

			const { total, dialog, loading, dictList, formData, rules, queryParams } =
			  toRefs(state);

			function handleQuery() {
			  context.emit('dictClick', null);
			  state.loading = true;
			  listPageDictTypes(state.queryParams).then(res=> {
				state.dictList = res.data;
				state.total = res.total;
				state.loading = false;
			  });
			}

			function resetQuery() {
			  queryFormRef.value.resetFields();
			  handleQuery();
			}

			function handleSelectionChange(selection) {
			  state.ids = selection.map((item) => item.id);
			  state.single = selection.length !== 1;
			  state.multiple = !selection.length;
			}

			function handleAdd() {
			  state.dialog = {
				title: '添加字典',
				visible: true,
			  };
			}

			function handleUpdate(row) {
			  state.dialog = {
				title: '修改字典',
				visible: true,
			  };
			  const id = row.id || state.ids;
			  getDictFormData(id).then(({ data }) => {
				state.formData = data;
			  });
			}

			function submitForm() {
			  dataFormRef.value.validate((isValid) => {
				if (isValid) {
				  if (state.formData.id) {
					updateDictType(state.formData.id, state.formData).then(() => {
					  ElMessage.success('修改成功');
					  cancel();
					  handleQuery();
					});
				  } else {
					addDictType(state.formData).then(() => {
					  ElMessage.success('新增成功');
					  cancel();
					  handleQuery();
					});
				  }
				}
			  });
			}

			function cancel() {
			  state.formData.id = undefined;
			  dataFormRef.value.resetFields();
			  state.dialog.visible = false;
			}

			function handleDelete(row) {
			  const ids = [row.id || state.ids].join(',');
			  ElMessageBox.confirm('确认删除已选中的数据项?', '警告', {
				confirmButtonText: '确定',
				cancelButtonText: '取消',
				type: 'warning',
			  })
				.then(() => {
				  deleteDictTypes(ids).then(() => {
					ElMessage.success('删除成功');
					handleQuery();
				  });
				})
				.catch(() => ElMessage.info('已取消删除'));
			}

			function handleRowClick(row) {
			  context.emit('dictClick', row);
			}

			onMounted(() => {
			  handleQuery();
			});
			return {handleQuery,handleDelete,handleRowClick,
			       cancel,submitForm,resetQuery,handleUpdate,handleAdd,handleSelectionChange,
				   //function
				   state,total, dialog, loading, dictList, formData, rules, queryParams,
				   //modal
				   queryFormRef,dataFormRef,
				   //ref
				
			}
  }
};
</script>