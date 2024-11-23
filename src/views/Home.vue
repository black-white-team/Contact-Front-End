<template>
  <div style="padding: 10px">
<!--    功能-->
    <div style="margin: 10px 0">
      <el-button type="primary" @click="add">New</el-button>
      <el-button type="primary" @click="">Import</el-button>
      <el-button type="primary" @click="">Export</el-button>
    </div>
<!--    搜索-->
    <div style="margin: 10px 0">
      <el-input v-model="search" placeholder="Enter content" style="width: 20%" clearable></el-input>
      <el-button type="primary" style="margin: 5px" @click="load">Search</el-button>
      <el-button type="primary" style="margin: 5px" @click="loadBookMark">BookMark</el-button>
    </div>
    <el-table :data="tableData" border stripe style="width: 100%">
      <el-table-column prop="id" label="ID" sortable />
      <el-table-column prop="username" label="Username" />
      <el-table-column prop="phonenumber" label="Phone Number" />
      <el-table-column prop="emailaddress" label="Email Address" />
      <el-table-column prop="socialmediahandles" label="Social Media Handles" />
      <el-table-column prop="physicaladdresses" label="Physical Address" />
      <el-table-column fixed="right" label="Operations" min-width="80">
        <template #default="scope">
          <el-button type="primary" size="small" @click="handleEdit(scope.row)">Edit</el-button>
          <el-popconfirm title="Confirm delete?" @confirm="handleDelete(scope.row.id)">
            <template #reference>
              <el-button size="small" type="danger">Delete</el-button>
            </template>
          </el-popconfirm>
          <el-button type="warning" circle @click="handleBookMark(scope.row)"><el-icon><Star /></el-icon></el-button>
        </template>
      </el-table-column>
    </el-table>

    <div style="padding: 10px">
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="currentPage"
        :page-sizes="[5, 10, 20]"
        :page-size="pageSize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination>

      <el-dialog v-model="dialogVisible" title="Tips" width="500">
        <el-form :model="form" label-width="160px">
          <el-form-item label="ID">
            <el-input v-model="form.id" style="width: 80%" />
          </el-form-item>
          <el-form-item label="Username">
            <el-input v-model="form.username" style="width: 80%" />
          </el-form-item>
          <el-form-item label="Phone Number">
            <el-input v-model="form.phonenumber" style="width: 80%" />
          </el-form-item>
          <el-form-item label="Email Address">
            <el-input type="textarea" v-model="form.emailaddress" style="width: 80%" />
          </el-form-item>
          <el-form-item label="Social Media Handles">
            <el-input type="textarea" v-model="form.socialmediahandles" style="width: 80%" />
          </el-form-item>
          <el-form-item label="Physical Address">
            <el-input type="textarea" v-model="form.physicaladdresses" style="width: 80%" />
          </el-form-item>
        </el-form>
        <template #footer>
          <div class="dialog-footer">
            <el-button @click="dialogVisible = false">Cancel</el-button>
            <el-button type="primary" @click="save">
              Confirm
            </el-button>
          </div>
        </template>
      </el-dialog>
    </div>
  </div>
</template>

<script>
import axios from "axios";
import {ElMessage} from "element-plus";
import { Star } from '@element-plus/icons-vue';

export default {
  name: 'Home',
  components: {
    Star,
  },

  data() {
    return {
      form: {},
      dialogVisible: false,
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
      })
    },

    loadBookMark() {
      axios.get("/api/bookmark", {
        params: {
          pageNum: this.currentPage,
          pageSize: this.pageSize,
          search: this.search,
        }
      }).then(res => {
        console.log(res)
        this.tableData = res.data.data.records
        this.total = res.data.data.total
      })
    },

    add() {
      this.dialogVisible = true
      this.form = {}
    },

    save() {
      // id 是数字类型，确保比较时不会误将 0 当作 falsy 值
      if (Number.isFinite(this.form.id) && this.form.id !== 0) { // 更新
        axios.put("/api/user", this.form).then(res => {
          console.log(res);
          if (res.data.code === '0') {
            ElMessage({
              type: 'success',
              message: 'Updated Successfully'
            });
          } else {
            ElMessage({
              type: 'error',
              message: res.data.msg
            });
          }
          this.load(); // 刷新表格数据
          this.dialogVisible = false; // 关闭弹窗
        });
      } else { // 新增
        axios.post("/api/user", this.form).then(res => {
          console.log(res);
          if (res.data.code === '0') {
            ElMessage({
              type: 'success',
              message: 'Added Successfully'
            });
          } else {
            ElMessage({
              type: 'error',
              message: res.data.msg
            });
          }
          this.load(); // 刷新表格数据
          this.dialogVisible = false; // 关闭弹窗
        });
      }
    },

    handleEdit(row) {
      this.form = JSON.parse(JSON.stringify(row))
      this.dialogVisible = true
    },

    handleDelete(id) {
      console.log(id)
      axios.delete("/api/user/" + id).then(res => {
        if (res.data.code === '0') {
          ElMessage({
            type: 'success',
            message: 'Deleted Successfully'
          });
        } else {
          ElMessage({
            type: 'error',
            message: res.data.msg
          });
        }
        this.load(); // 刷新表格数据
        this.dialogVisible = false; // 关闭弹窗
      });
    },

    handleBookMark(id) {
      axios.post("/api/bookmark", { id: id }).then(res => {
        console.log(res);
        if (res.data.code === '0') {
          ElMessage({
            type: 'success',
            message: 'Added Successfully'
          });
        } else {
          ElMessage({
            type: 'error',
            message: res.data.msg
          });
        }
        this.loadBookMark(); // 刷新表格数据
        this.dialogVisible = false; // 关闭弹窗
      });
    },

    handleSizeChange(pageSize) { // 改变当前每页的个数触发
      this.pageSize = pageSize
      this.load()
    },

    handleCurrentChange(pageNum) { // 改变当前页码触发
      this.currentPage = pageNum
      this.load()
    }
  }
}
</script>
