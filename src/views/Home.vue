<template>
  <div class="container">
    <!-- 功能按钮 -->
    <div class="button-group">
      <el-button class="custom-button success" @click="add">New</el-button>
      <el-button class="custom-button info" @click="openImportDialog"
        >Import</el-button
      >
      <el-button class="custom-button info" @click="exportExcel"
        >Export</el-button
      >
    </div>

    <!-- 搜索区域 -->
    <div class="search-group">
      <el-input
        v-model="search"
        placeholder="Enter content"
        class="search-input"
        clearable
      ></el-input>
      <el-button class="custom-button primary" @click="load">Search</el-button>
      <el-button class="custom-button success" @click="loadBookMark"
        >BookMark</el-button
      >
    </div>

    <!-- 数据表格 -->
    <el-table :data="tableData" border stripe class="custom-table">
      <el-table-column prop="username" label="Username" width="150" />

      <!-- 电话号码列 -->
      <el-table-column label="Phone Numbers" width="250">
        <template #default="scope">
          <div v-for="phone in scope.row.phoneNumbers" :key="phone.id">
            {{ phone.type }}: {{ phone.number }}
          </div>
        </template>
      </el-table-column>

      <!-- 电子邮件地址列 -->
      <el-table-column label="Email Addresses" width="250">
        <template #default="scope">
          <div v-for="email in scope.row.emailAddresses" :key="email.id">
            {{ email.type }}: {{ email.email }}
          </div>
        </template>
      </el-table-column>

      <!-- 社交媒体账户列 -->
      <el-table-column label="Social Media Handles" width="250">
        <template #default="scope">
          <div v-for="social in scope.row.socialMediaHandles" :key="social.id">
            {{ social.platform }}: {{ social.handle }}
          </div>
        </template>
      </el-table-column>

      <!-- 物理地址列 -->
      <el-table-column label="Physical Addresses" width="250">
        <template #default="scope">
          <div v-for="address in scope.row.physicalAddresses" :key="address.id">
            {{ address.type }}: {{ address.address }}
          </div>
        </template>
      </el-table-column>

      <el-table-column fixed="right" label="Operations" min-width="180">
        <template #default="scope">
          <el-button
            class="operation-button modify-button"
            size="mini"
            type="warning"
            @click="handleEdit(scope.row)"
          >
            Edit
          </el-button>
          <el-popconfirm
            title="Confirm delete?"
            @confirm="handleDelete(scope.row.id)"
          >
            <template #reference>
              <el-button
                class="operation-button delete-button"
                size="mini"
                type="danger"
              >
                Delete
              </el-button>
            </template>
          </el-popconfirm>
          <el-button
            class="operation-button"
            :type="scope.row.bookmark ? 'warning' : 'default'"
            circle
            @click="handleBookMark(scope.row)"
          >
            <el-icon :class="{ bookmarked: scope.row.bookmark }"
              ><Star
            /></el-icon>
          </el-button>
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
        :total="total"
      >
      </el-pagination>
    </div>

    <!-- 编辑/新增对话框 -->
    <el-dialog
      v-model="dialogVisible"
      :title="form.id ? 'Edit User' : 'Add User'"
      width="800px"
      :close-on-click-modal="false"
    >
      <el-form :model="form" label-width="180px" class="custom-form">
        <el-form-item label="Username">
          <el-input v-model="form.username" class="form-input" />
        </el-form-item>

        <!-- 电话号码 -->
        <el-form-item label="Phone Numbers">
          <el-table :data="form.phoneNumbers" style="width: 100%">
            <el-table-column
              prop="type"
              label="Type"
              width="150"
            ></el-table-column>
            <el-table-column
              prop="number"
              label="Number"
              width="200"
            ></el-table-column>
            <el-table-column label="Operations" width="250">
              <template #default="scope">
                <div class="operation-group">
                  <el-button
                    size="default"
                    class="modify-button"
                    type="warning"
                    @click="
                      openContactDialog('phoneNumbers', scope.$index, scope.row)
                    "
                  >
                    Modify
                  </el-button>
                  <el-button
                    size="default"
                    class="delete-button"
                    type="danger"
                    @click="deleteContactItem('phoneNumbers', scope.$index)"
                  >
                    Delete
                  </el-button>
                </div>
              </template>
            </el-table-column>
          </el-table>
          <el-button
            type="success"
            size="small"
            class="add-button"
            @click="openContactDialog('phoneNumbers')"
          >
            Add Phone
          </el-button>
        </el-form-item>

        <!-- 电子邮件地址 -->
        <el-form-item label="Email Addresses">
          <el-table :data="form.emailAddresses" style="width: 100%">
            <el-table-column
              prop="type"
              label="Type"
              width="150"
            ></el-table-column>
            <el-table-column
              prop="email"
              label="Email"
              width="200"
            ></el-table-column>
            <el-table-column label="Operations" width="250">
              <template #default="scope">
                <div class="operation-group">
                  <el-button
                    size="mini"
                    class="modify-button"
                    type="warning"
                    @click="
                      openContactDialog(
                        'emailAddresses',
                        scope.$index,
                        scope.row
                      )
                    "
                  >
                    Modify
                  </el-button>
                  <el-button
                    size="mini"
                    class="delete-button"
                    type="danger"
                    @click="deleteContactItem('emailAddresses', scope.$index)"
                  >
                    Delete
                  </el-button>
                </div>
              </template>
            </el-table-column>
          </el-table>
          <el-button
            type="success"
            size="small"
            class="add-button"
            @click="openContactDialog('emailAddresses')"
          >
            Add Email
          </el-button>
        </el-form-item>

        <!-- 社交媒体账户 -->
        <el-form-item label="Social Media Handles">
          <el-table :data="form.socialMediaHandles" style="width: 100%">
            <el-table-column
              prop="platform"
              label="Platform"
              width="150"
            ></el-table-column>
            <el-table-column
              prop="handle"
              label="Handle"
              width="200"
            ></el-table-column>
            <el-table-column label="Operations" width="250">
              <template #default="scope">
                <div class="operation-group">
                  <el-button
                    size="mini"
                    class="modify-button"
                    type="warning"
                    @click="
                      openContactDialog(
                        'socialMediaHandles',
                        scope.$index,
                        scope.row
                      )
                    "
                  >
                    Modify
                  </el-button>
                  <el-button
                    size="mini"
                    class="delete-button"
                    type="danger"
                    @click="
                      deleteContactItem('socialMediaHandles', scope.$index)
                    "
                  >
                    Delete
                  </el-button>
                </div>
              </template>
            </el-table-column>
          </el-table>
          <el-button
            type="success"
            size="small"
            class="add-button"
            @click="openContactDialog('socialMediaHandles')"
          >
            Add Social Media
          </el-button>
        </el-form-item>

        <!-- 物理地址 -->
        <el-form-item label="Physical Addresses">
          <el-table :data="form.physicalAddresses" style="width: 100%">
            <el-table-column
              prop="type"
              label="Type"
              width="150"
            ></el-table-column>
            <el-table-column
              prop="address"
              label="Address"
              width="200"
            ></el-table-column>
            <el-table-column label="Operations" width="250">
              <template #default="scope">
                <div class="operation-group">
                  <el-button
                    size="mini"
                    class="modify-button"
                    type="warning"
                    @click="
                      openContactDialog(
                        'physicalAddresses',
                        scope.$index,
                        scope.row
                      )
                    "
                  >
                    Modify
                  </el-button>
                  <el-button
                    size="mini"
                    class="delete-button"
                    type="danger"
                    @click="
                      deleteContactItem('physicalAddresses', scope.$index)
                    "
                  >
                    Delete
                  </el-button>
                </div>
              </template>
            </el-table-column>
          </el-table>
          <el-button
            type="success"
            size="small"
            class="add-button"
            @click="openContactDialog('physicalAddresses')"
          >
            Add Address
          </el-button>
        </el-form-item>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button class="footer-button" @click="dialogVisible = false"
            >Cancel</el-button
          >
          <el-button class="footer-button primary" type="primary" @click="save">
            Confirm
          </el-button>
        </div>
      </template>
    </el-dialog>

    <!-- 联系方式对话框 -->
    <el-dialog
      v-model="contactDialogVisible"
      :title="currentContactTypeTitle"
      width="600px"
      :close-on-click-modal="false"
    >
      <el-form
        :model="currentContactData"
        label-width="120px"
        class="custom-form"
      >
        <template v-if="currentContactType === 'phoneNumbers'">
          <el-form-item label="Type">
            <el-select
              v-model="currentContactData.type"
              placeholder="Select Type"
            >
              <el-option label="Mobile" value="mobile"></el-option>
              <el-option label="Home" value="home"></el-option>
              <el-option label="Work" value="work"></el-option>
              <el-option label="Other" value="other"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="Phone Number">
            <el-input
              v-model="currentContactData.number"
              placeholder="Enter phone number"
            ></el-input>
          </el-form-item>
        </template>

        <template v-if="currentContactType === 'emailAddresses'">
          <el-form-item label="Type">
            <el-select
              v-model="currentContactData.type"
              placeholder="Select Type"
            >
              <el-option label="Personal" value="personal"></el-option>
              <el-option label="Work" value="work"></el-option>
              <el-option label="Other" value="other"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="Email Address">
            <el-input
              v-model="currentContactData.email"
              placeholder="Enter email address"
            ></el-input>
          </el-form-item>
        </template>

        <template v-if="currentContactType === 'socialMediaHandles'">
          <el-form-item label="Platform">
            <el-select
              v-model="currentContactData.platform"
              placeholder="Select Platform"
            >
              <el-option label="Twitter" value="Twitter"></el-option>
              <el-option label="Facebook" value="Facebook"></el-option>
              <el-option label="LinkedIn" value="LinkedIn"></el-option>
              <el-option label="Instagram" value="Instagram"></el-option>
              <el-option label="Other" value="Other"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="Handle">
            <el-input
              v-model="currentContactData.handle"
              placeholder="Enter handle"
            ></el-input>
          </el-form-item>
        </template>

        <template v-if="currentContactType === 'physicalAddresses'">
          <el-form-item label="Type">
            <el-select
              v-model="currentContactData.type"
              placeholder="Select Type"
            >
              <el-option label="Home" value="home"></el-option>
              <el-option label="Work" value="work"></el-option>
              <el-option label="Other" value="other"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="Address">
            <el-input
              v-model="currentContactData.address"
              placeholder="Enter physical address"
            ></el-input>
          </el-form-item>
        </template>
      </el-form>
      <template #footer>
        <div class="dialog-footer">
          <el-button
            class="footer-button"
            @click="contactDialogVisible = false"
          >
            Cancel
          </el-button>
          <el-button
            class="footer-button primary"
            type="primary"
            @click="confirmContact"
          >
            Confirm
          </el-button>
        </div>
      </template>
    </el-dialog>

    <!-- 导入对话框 -->
    <el-dialog
      title="Import Excel"
      v-model="importDialogVisible"
      width="500px"
      :close-on-click-modal="false"
    >
      <el-upload
        class="upload-demo"
        drag
        action="/api/user/import"
        :show-file-list="false"
        accept=".xlsx,.xls"
        :before-upload="beforeUpload"
        :on-success="handleImportSuccess"
        :on-error="handleImportError"
      >
        <i class="el-icon-upload"></i>
        <div class="el-upload__text">
          Drag Excel files here or click to upload
        </div>
        <div class="el-upload__tip">Only .xlsx and .xls files are allowed</div>
      </el-upload>
      <template #footer>
        <div class="dialog-footer">
          <el-button class="footer-button" @click="importDialogVisible = false">
            Cancel
          </el-button>
        </div>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import axios from "axios";
import { ElMessage } from "element-plus";
import { Star } from "@element-plus/icons-vue";

export default {
  name: "Home",
  components: {
    Star,
  },

  data() {
    return {
      form: {
        id: "",
        username: "",
        bookmark: false, // 新增 bookmark 字段
        phoneNumbers: [],
        emailAddresses: [],
        socialMediaHandles: [],
        physicalAddresses: [],
      },
      dialogVisible: false,
      importDialogVisible: false,
      search: "",
      currentPage: 1,
      pageSize: 10,
      total: 0,
      tableData: [],

      // 联系方式对话框相关
      contactDialogVisible: false,
      currentContactType: "", // 'phoneNumbers', 'emailAddresses', 'socialMediaHandles', 'physicalAddresses'
      currentContactData: {},
      currentContactIndex: null, // 用于修改时的索引
    };
  },

  created() {
    this.load();
  },

  methods: {
    // 加载用户数据
    load() {
      axios
        .get("/api/user", {
          params: {
            pageNum: this.currentPage,
            pageSize: this.pageSize,
            search: this.search,
          },
        })
        .then((res) => {
          console.log(res);
          this.tableData = res.data.data.records;
          this.total = res.data.data.total;
        })
        .catch((error) => {
          ElMessage.error("Failed to load data");
          console.error(error);
        });
    },

    // 加载书签数据
    loadBookMark() {
      axios
        .get("/api/user/favorites", {
          params: {
            pageNum: this.currentPage,
            pageSize: this.pageSize,
            search: this.search,
          },
        })
        .then((res) => {
          console.log(res);
          this.tableData = res.data.data.records;
          this.total = res.data.data.total;
        })
        .catch((error) => {
          ElMessage.error("Failed to load bookmarks");
          console.error(error);
        });
    },

    // 新增用户
    add() {
      this.dialogVisible = true;
      this.form = {
        id: "",
        username: "",
        bookmark: false, // 重置 bookmark 字段
        phoneNumbers: [],
        emailAddresses: [],
        socialMediaHandles: [],
        physicalAddresses: [],
      };
    },

    // 保存用户（新增或更新）
    save() {
      if (this.form.id && Number.isFinite(Number(this.form.id))) {
        // 更新
        axios
          .put(`/api/user/${this.form.id}`, this.form)
          .then((res) => {
            console.log(res);
            if (res.data.code === "0") {
              ElMessage.success("Updated Successfully");
            } else {
              ElMessage.error(res.data.msg);
            }
            this.load(); // 刷新表格数据
            this.dialogVisible = false; // 关闭弹窗
          })
          .catch((error) => {
            ElMessage.error("Update failed");
            console.error(error);
          });
      } else {
        // 新增
        axios
          .post("/api/user", this.form)
          .then((res) => {
            console.log(res);
            if (res.data.code === "0") {
              ElMessage.success("Added Successfully");
            } else {
              ElMessage.error(res.data.msg);
            }
            this.load(); // 刷新表格数据
            this.dialogVisible = false; // 关闭弹窗
          })
          .catch((error) => {
            ElMessage.error("Addition failed");
            console.error(error);
          });
      }
    },

    // 编辑用户
    handleEdit(row) {
      this.form = { ...row };
      this.dialogVisible = true;
    },

    // 删除用户
    handleDelete(id) {
      axios
        .delete(`/api/user/${id}`)
        .then((res) => {
          if (res.data.code === "0") {
            ElMessage.success("Deleted Successfully");
          } else {
            ElMessage.error(res.data.msg);
          }
          this.load(); // 刷新表格数据
        })
        .catch((error) => {
          ElMessage.error("Deletion failed");
          console.error(error);
        });
    },

    // 添加书签
    handleBookMark(row) {
      axios
        .patch(`/api/user/favorite/${row.id}`)
        .then((res) => {
          console.log(res);
          if (res.data.code === "0") {
            ElMessage.success(
              row.bookmark ? "Removed Bookmark" : "Bookmarked Successfully"
            );
          } else {
            ElMessage.error(res.data.msg);
          }
          // 刷新表格数据
          this.load();
        })
        .catch((error) => {
          ElMessage.error("Bookmark failed");
          console.error(error);
        });
    },

    // 分页改变每页数量
    handleSizeChange(pageSize) {
      this.pageSize = pageSize;
      this.load();
    },

    // 分页改变当前页码
    handleCurrentChange(pageNum) {
      this.currentPage = pageNum;
      this.load();
    },

    // EXCEL 导出
    exportExcel() {
      axios({
        method: "get",
        url: "/api/user/export",
        responseType: "blob",
      })
        .then((res) => {
          const blob = new Blob([res.data], {
            type: "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet;charset=utf-8",
          });
          const downloadElement = document.createElement("a");
          const href = window.URL.createObjectURL(blob);
          downloadElement.href = href;
          downloadElement.download = "users_with_contacts.xlsx";
          document.body.appendChild(downloadElement);
          downloadElement.click();
          document.body.removeChild(downloadElement);
          window.URL.revokeObjectURL(href);
        })
        .catch((error) => {
          ElMessage.error("Export failed");
          console.error(error);
        });
    },

    // 打开导入对话框
    openImportDialog() {
      this.importDialogVisible = true;
    },

    // 验证上传文件
    beforeUpload(file) {
      const isExcel =
        file.type ===
          "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet" ||
        file.type === "application/vnd.ms-excel";

      if (!isExcel) {
        ElMessage.error("请上传 Excel 文件");
      }
      const isLt2M = file.size / 1024 / 1024 < 2;
      if (!isLt2M) {
        ElMessage.error("文件大小必须小于 2MB");
      }
      return isExcel && isLt2M;
    },

    // 上传成功回调
    handleImportSuccess(response, file, fileList) {
      ElMessage.success("Import successful");
      this.importDialogVisible = false;
      this.load(); // 刷新数据
    },

    // 上传失败回调
    handleImportError(err, file, fileList) {
      ElMessage.error("Import failed");
      console.error(err);
    },

    // 打开联系方式对话框
    openContactDialog(type, index = null, row = {}) {
      this.currentContactType = type;
      this.currentContactIndex = index;
      this.currentContactData =
        index !== null ? { ...row } : this.getEmptyContactData(type);
      this.contactDialogVisible = true;
    },

    // 获取空的联系方式数据
    getEmptyContactData(type) {
      switch (type) {
        case "phoneNumbers":
          return { type: "", number: "" };
        case "emailAddresses":
          return { type: "", email: "" };
        case "socialMediaHandles":
          return { platform: "", handle: "" };
        case "physicalAddresses":
          return { type: "", address: "" };
        default:
          return {};
      }
    },

    // 确认联系方式操作
    confirmContact() {
      const type = this.currentContactType;
      const data = this.currentContactData;

      if (!this.validateContactData(type, data)) {
        ElMessage.error("Please fill in all required fields.");
        return;
      }

      if (this.currentContactIndex !== null) {
        // 修改现有数据
        this.form[type].splice(this.currentContactIndex, 1, data);
      } else {
        // 添加新数据
        this.form[type].push(data);
      }

      this.contactDialogVisible = false;
      ElMessage.success(
        `${this.getContactTypeLabel(type)} saved successfully.`
      );
    },

    // 验证联系方式数据
    validateContactData(type, data) {
      switch (type) {
        case "phoneNumbers":
          return data.type && data.number;
        case "emailAddresses":
          return data.type && data.email;
        case "socialMediaHandles":
          return data.platform && data.handle;
        case "physicalAddresses":
          return data.type && data.address;
        default:
          return false;
      }
    },

    // 获取联系方式类型的标签
    getContactTypeLabel(type) {
      switch (type) {
        case "phoneNumbers":
          return "Phone Number";
        case "emailAddresses":
          return "Email Address";
        case "socialMediaHandles":
          return "Social Media Handle";
        case "physicalAddresses":
          return "Physical Address";
        default:
          return "";
      }
    },

    // 删除联系方式项
    deleteContactItem(type, index) {
      this.form[type].splice(index, 1);
      ElMessage.success(
        `${this.getContactTypeLabel(type)} deleted successfully.`
      );
    },
  },
};
</script>

<style scoped>
/* 通用容器样式 */
.container {
  padding: 20px;
  background-color: #f5f5f7; /* 苹果风格浅灰色背景 */
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto,
    "Helvetica Neue", Arial, sans-serif;
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
  padding: 12px 24px; /* 增大内边距 */
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

.custom-button.warning {
  background-color: #ffcc00;
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
  padding: 8px 12px; /* Reduced padding for compactness */
}

/* 操作按钮 */
.operation-button {
  border-radius: 6px;
  font-size: 12px;
  padding: 6px 12px;
  margin-right: 5px;
  transition: background-color 0.3s, box-shadow 0.3s;
}

.operation-button.modify-button {
  background-color: #ffcc00; /* Yellow color for modify */
  color: #fff;
  border: none;
}

.operation-button.delete-button {
  background-color: #ff3b30;
  color: #fff;
  border: none;
}

.operation-button:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* 收藏图标样式 */
.bookmarked {
  color: #ffd700; /* 深色星标 (Gold) */
}
/* 操作按钮组布局 */
.operation-group {
  display: flex;
  gap: 5px;
  align-items: center;
}

/* 分页样式 */
.pagination-group {
  display: flex;
  justify-content: flex-end;
  margin-top: 20px;
}

/* 对话框样式 */
.custom-form {
  padding: 10px 0; /* Reduced padding for compactness */
}

.form-input {
  border-radius: 6px;
}

.el-dialog__header {
  border-bottom: none;
}

.el-dialog__body {
  padding: 10px 20px; /* Reduced padding */
}

.el-dialog__footer {
  padding: 10px 20px; /* Reduced padding */
}

.dialog-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
}

.footer-button {
  border-radius: 6px;
  padding: 6px 12px; /* Reduced padding */
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
  padding: 30px 0; /* Reduced padding */
  border: 2px dashed #007aff;
  border-radius: 12px;
  transition: border-color 0.3s;
}

.upload-demo:hover {
  border-color: #0a84ff;
}

.upload-demo i {
  font-size: 24px; /* Reduced icon size */
  color: #0a84ff;
}

.upload-demo .el-upload__text {
  margin-top: 8px; /* Reduced margin */
  font-size: 14px;
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

/* 联系方式对话框样式 */
.dynamic-field {
  display: flex;
  align-items: center;
  margin-bottom: 10px;
}

.dynamic-field .el-input,
.dynamic-field .el-select {
  flex: 1;
  margin-right: 10px;
}

.dynamic-field .el-button {
  margin-left: 5px;
}

/* 添加按钮样式 */
.add-button {
  margin-top: 8px; /* Reduced margin */
}
</style>
