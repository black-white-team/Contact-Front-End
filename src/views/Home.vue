<template>
  <div class="container">
    <!-- 功能按钮 -->
    <div class="button-group">
      <el-button class="custom-button success" @click="add">New</el-button>
      <el-button class="custom-button info" @click="openImportDialog">Import</el-button>
      <el-button class="custom-button info" @click="exportExcel">Export</el-button>
    </div>

    <!-- 搜索区域 -->
    <div class="search-group">
      <el-input v-model="search" placeholder="Enter content" class="search-input" clearable></el-input>
      <el-button class="custom-button primary" @click="load">Search</el-button>
      <el-button class="custom-button success" @click="loadBookMark">BookMark</el-button>
    </div>

    <!-- 数据表格 -->
    <el-table :data="tableData" border stripe class="custom-table">
      <el-table-column prop="id" label="ID" sortable />
      <el-table-column prop="username" label="Username" />
      <el-table-column prop="phonenumber" label="Phone Number" />
      <el-table-column prop="emailaddress" label="Email Address" />
      <el-table-column prop="socialmediahandles" label="Social Media Handles" />
      <el-table-column prop="physicaladdresses" label="Physical Address" />
      <el-table-column fixed="right" label="Operations" min-width="150">
        <template #default="scope">
          <el-button class="operation-button" type="primary" size="small" @click="handleEdit(scope.row)">Edit</el-button>
          <el-popconfirm title="Confirm delete?" @confirm="handleDelete(scope.row.id)">
            <template #reference>
              <el-button class="operation-button" size="small" type="danger">Delete</el-button>
            </template>
          </el-popconfirm>
          <el-button class="operation-button" type="warning" circle @click="handleBookMark(scope.row)">
            <el-icon><Star /></el-icon>
          </el-button>
        </template>
      </el-table-column>
      <el-table-column label="Bookmarked" width="120">
        <template #default="scope">
          <el-tag v-if="scope.row.bookmarks" effect="dark" type="success">Yes</el-tag>
          <el-tag v-else effect="dark" type="info">No</el-tag>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页 -->
    <div class="pagination-group">
      <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="currentPage"
          :page-sizes="[5, 10, 20]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
      </el-pagination>
    </div>

    <!-- 编辑/新增对话框 -->
    <el-dialog v-model="dialogVisible" title="User Information" width="500px" :close-on-click-modal="false">
      <el-form :model="form" label-width="160px" class="custom-form">
        <el-form-item label="ID">
          <el-input v-model="form.id" class="form-input" />
        </el-form-item>
        <el-form-item label="Username">
          <el-input v-model="form.username" class="form-input" />
        </el-form-item>
        <el-form-item label="Phone Number">
          <el-input v-model="form.phonenumber" class="form-input" />
        </el-form-item>
        <el-form-item label="Email Address">
          <el-input type="textarea" v-model="form.emailaddress" class="form-input" />
        </el-form-item>
        <el-form-item label="Social Media Handles">
          <el-input type="textarea" v-model="form.socialmediahandles" class="form-input" />
        </el-form-item>
        <el-form-item label="Physical Address">
          <el-input type="textarea" v-model="form.physicaladdresses" class="form-input" />
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button class="footer-button" @click="dialogVisible = false">Cancel</el-button>
          <el-button class="footer-button primary" type="primary" @click="save">Confirm</el-button>
        </div>
      </template>
    </el-dialog>

    <!-- 导入对话框 -->
    <el-dialog
        title="Import Excel"
        v-model="importDialogVisible"
        width="500px"
        :close-on-click-modal="false">
      <el-upload
          class="upload-demo"
          drag
          action="/api/user/import"
          :show-file-list="false"
          accept=".xlsx,.xls"
          :before-upload="beforeUpload"
          :on-success="handleImportSuccess"
          :on-error="handleImportError">
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">Drag Excel files here or click to upload</div>
        <div slot="tips" class="el-upload__tip">Only .xlsx and .xls files are allowed</div>
      </el-upload>
      <template #footer>
        <div class="dialog-footer">
          <el-button class="footer-button" @click="importDialogVisible = false">Cancel</el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import axios from "axios";
import { ElMessage } from "element-plus";
import { Star } from '@element-plus/icons-vue';

export default {
  name: 'Home',
  components: {
    Star,
  },

  data() {
    return {
      form: {
        id: '',
        username: '',
        phonenumber: '',
        emailaddress: '',
        socialmediahandles: '',
        physicaladdresses: ''
      },
      dialogVisible: false,
      importDialogVisible: false, // 控制导入对话框的显示
      search: '',
      currentPage: 1,
      pageSize: 10,
      total: 0,
      tableData: []
    }
  },

  created() {
    this.load()
  },

  methods: {
    // 加载用户数据
    load() {
      axios.get("/api/user", {
        params: {
          pageNum: this.currentPage,
          pageSize: this.pageSize,
          search: this.search,
        }
      }).then(res => {
        console.log(res)
        this.tableData = res.data.data.records
        this.total = res.data.data.total
      }).catch(error => {
        ElMessage.error('Failed to load data');
        console.error(error);
      })
    },

    // 加载书签数据
    loadBookMark() {
      axios.get("/api/user/favorites", {
        params: {
          pageNum: this.currentPage,
          pageSize: this.pageSize,
        }
      }).then(res => {
        if (res.data.code === '0' && res.data.data) {
          this.tableData = res.data.data.records; // 确保这里获取的是数据列表
          this.total = res.data.data.total; // 更新总数为返回的数据的总数
        } else {
          ElMessage.error(res.data.msg || 'Failed to load bookmarks');
        }
      }).catch(error => {
        ElMessage.error('Failed to load bookmarks');
        console.error(error);
      });
    },

    // 新增用户
    add() {
      this.dialogVisible = true
      this.form = {
        id: '',
        username: '',
        phonenumber: '',
        emailaddress: '',
        socialmediahandles: '',
        physicaladdresses: '',
        bookmarks: ''
      }
    },

    // 保存用户（新增或更新）
    save() {
      // 检查form.id是否存在且不为0
      if (this.form.id && Number.isFinite(this.form.id) && this.form.id !== 0) {
        // 更新
        axios.put("/api/user", this.form).then(res => {
          console.log(res);
          if (res.data.code === '0') {
            ElMessage.success('Updated Successfully');
          } else {
            ElMessage.error(res.data.msg);
          }
          this.load(); // 刷新表格数据
          this.dialogVisible = false; // 关闭弹窗
        }).catch(error => {
          ElMessage.error('Update failed');
          console.error(error);
        });
      } else {
        // 新增
        axios.post("/api/user", this.form).then(res => {
          console.log(res);
          if (res.data.code === '0') {
            ElMessage.success('Added Successfully');
          } else {
            ElMessage.error(res.data.msg);
          }
          this.load(); // 刷新表格数据
          this.dialogVisible = false; // 关闭弹窗
        }).catch(error => {
          ElMessage.error('Addition failed');
          console.error(error);
        });
      }
    },

    // 编辑用户
    handleEdit(row) {
      this.form = { ...row }
      this.dialogVisible = true
    },

    // 删除用户
    handleDelete(id) {
      axios.delete(`/api/user/${id}`).then(res => {
        if (res.data.code === '0') {
          ElMessage.success('Deleted Successfully');
        } else {
          ElMessage.error(res.data.msg);
        }
        this.load(); // 刷新表格数据
      }).catch(error => {
        ElMessage.error('Deletion failed');
        console.error(error);
      });
    },

    // 添加书签
    handleBookMark(row) {
      axios.patch(`/api/user/favorite/${row.id}`).then(res => {
        console.log(res);
        if (res.data.code === '0') {
          ElMessage.success('Bookmark toggled Successfully');
          this.load(); // 刷新表格数据
        } else {
          ElMessage.error(res.data.msg);
        }
      }).catch(error => {
        ElMessage.error('Bookmark failed');
        console.error(error);
      });
    },

    // 分页改变每页数量
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      this.load()
    },

    // 分页改变当前页码
    handleCurrentChange(pageNum) {
      this.currentPage = pageNum
      this.load()
    },

    // EXCEL 导出
    exportExcel() {
      axios({
        method: 'get',
        url: '/api/user/export',
        responseType: 'blob'
      }).then(res => {
        const blob = new Blob([res.data], { type: 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8' });
        const downloadElement = document.createElement('a');
        const href = window.URL.createObjectURL(blob);
        downloadElement.href = href;
        downloadElement.download = 'users.xlsx';
        document.body.appendChild(downloadElement);
        downloadElement.click();
        document.body.removeChild(downloadElement);
        window.URL.revokeObjectURL(href);
      }).catch(error => {
        ElMessage.error('Export failed');
        console.error(error);
      });
    },

    // 打开导入对话框
    openImportDialog() {
      this.importDialogVisible = true;
    },

    // 验证上传文件
    beforeUpload(file) {
      const isExcel = file.type === 'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet' ||
          file.type === 'application/vnd.ms-excel';
      if (!isExcel) {
        ElMessage.error('请上传 Excel 文件');
      }
      const isLt2M = file.size / 1024 / 1024 < 2;
      if (!isLt2M) {
        ElMessage.error('文件大小必须小于 2MB');
      }
      return isExcel && isLt2M;
    },

    // 上传成功回调
    handleImportSuccess(response, file, fileList) {
      ElMessage.success('Import successful');
      this.importDialogVisible = false;
      this.load(); // 刷新数据
    },

    // 上传失败回调
    handleImportError(err, file, fileList) {
      ElMessage.error('Import failed');
      console.error(err);
    },
  }
}
</script>

<style scoped>
/* 通用容器样式 */
.container {
  padding: 20px;
  background-color: #f5f5f7; /* 苹果风格浅灰色背景 */
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
  color: #333;
}

/* 功能按钮组 */
.button-group {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

/* 按钮样式 */
.custom-button {
  border-radius: 8px;
  padding: 8px 16px;
  font-size: 14px;
  font-weight: 500;
  transition: background-color 0.3s, box-shadow 0.3s;
}

.custom-button.success {
  background-color: #34c759;
  color: #fff;
  border: none;
}

.custom-button.info {
  background-color: #17a2b8;
  color: #fff;
  border: none;
}

.custom-button.primary {
  background-color: #0a84ff;
  color: #fff;
  border: none;
}

.custom-button:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

/* 搜索区域 */
.search-group {
  display: flex;
  align-items: center;
  gap: 10px;
  margin-bottom: 20px;
}

.search-input {
  flex: 1;
  max-width: 300px;
}

/* 表格样式 */
.custom-table {
  background-color: #fff;
  border-radius: 12px;
  overflow: hidden;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.05);
}

.el-table th {
  background-color: #f0f0f5;
  color: #555;
  font-weight: 600;
}

.el-table td {
  color: #666;
}

.el-table .cell {
  padding: 12px 16px;
}

/* 操作按钮 */
.operation-button {
  border-radius: 6px;
  font-size: 12px;
  padding: 6px 12px;
  margin-right: 5px;
  transition: background-color 0.3s, box-shadow 0.3s;
}

.operation-button.primary {
  background-color: #0a84ff;
  color: #fff;
  border: none;
}

.operation-button.danger {
  background-color: #ff3b30;
  color: #fff;
  border: none;
}

.operation-button.warning {
  background-color: #ffcc00;
  color: #fff;
  border: none;
}

.operation-button:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* 分页样式 */
.pagination-group {
  display: flex;
  justify-content: flex-end;
  margin-top: 20px;
}

/* 对话框样式 */
.custom-form {
  padding: 20px 0;
}

.form-input {
  border-radius: 6px;
}

.el-dialog__header {
  border-bottom: none;
}

.el-dialog__body {
  padding: 20px;
}

.el-dialog__footer {
  padding: 20px;
  border-top: none;
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.footer-button {
  border-radius: 6px;
  padding: 8px 16px;
  font-size: 14px;
  transition: background-color 0.3s, box-shadow 0.3s;
}

.footer-button.primary {
  background-color: #0a84ff;
  color: #fff;
  border: none;
}

.footer-button:hover {
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}

/* 上传区域样式 */
.upload-demo {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 40px 0;
  border: 2px dashed #007aff;
  border-radius: 12px;
  transition: border-color 0.3s;
}

.upload-demo:hover {
  border-color: #0a84ff;
}

.upload-demo i {
  font-size: 28px;
  color: #0a84ff;
}

.upload-demo .el-upload__text {
  margin-top: 10px;
  font-size: 16px;
  color: #555;
}

.upload-demo .el-upload__tip {
  font-size: 12px;
  color: #999;
}

/* 表单标签样式 */
.el-form-item__label {
  font-weight: 500;
  color: #333;
}
</style>