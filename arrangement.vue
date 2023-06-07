<template>
  <div>
    <div style="margin-bottom: 20px">
      <Query
        size="mini"
        labelWidth="100px"
        :itemWidth="7"
        :searchData="searchData"
        :searchForm="searchForm"
        :searchHandle="searchHandle"
        :list="list"
      ></Query>
    </div>
    <table>
      <!-- Table structure -->
      <thead>
        <tr>
          <th>日期</th>
          <th>版本/工单</th>
          <th>工作角色</th>
          <th>工作量（小时）</th>
          <th>操作</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(row, index) in tableData" :key="index">
          <td style="width: 300px">
            <el-date-picker
              v-model="row.scheduleDate"
              type="date"
              format="yyyy-MM-dd"
              v-if="row.isEditing"
              @change="handleDateChange(row)"
            ></el-date-picker>
            <span v-else>{{ row.scheduleDate }}</span>
          </td>

          <td style="width: 300px">
            <el-select
              v-model="row.versionOrissueShow"
              clearable
              filterable
              default-first-option
              v-if="row.isEditing"
            >
              <el-option-group
                v-for="group in options"
                :key="group.text"
                :label="group.text"
                placement="top"
              >
                <el-option
                  v-for="item in group.options"
                  :key="item.value"
                  :label="item.text"
                  :value="{ value: item.value, type: group.text }"
                >
                </el-option>
              </el-option-group>
            </el-select>
            <span v-else>{{ row.versionOrissueShow }}</span>
          </td>
          <td style="width: 300px">
            <el-input
              v-if="row.isEditing"
              v-model="role"
              placeholder="请输入工作角色"
              :readonly="true"
            ></el-input>
            <span v-else>{{ role }}</span>
          </td>
          <td style="width: 300px">
            <el-input
              v-if="row.isEditing"
              v-model="row.workload"
              placeholder="请输入工作量（小时）"
            />
            <span v-else>{{ row.workload }}</span>
          </td>
          <td>
            <button @click="handleEdit(row)">编辑</button>
            <button @click="handleEditSave(row)">保存</button>
            <button @click="handleDelete(row)">删除</button>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>
<script>
export default {
  name: "Department",
  data() {
    return {
      // tableData: [{isEditing:false},{isEditing:false}],
      tableData: [],
      options: [],
      role: "",
      searchForm: [
        {
          type: "Select",
          label: "经办人",
          prop: "member",
          placeholder: "请选择",
          clearable: true,
          change: (value) => {
            this.searchData.memberId = value;
          },
        },
      ],
      searchHandle: [
        {
          label: "查询",
          type: "primary",
          handle: () => {
            if (this.searchData.memberId.length === 0) {
              this.searchData.memberId = JSON.parse(
                localStorage.getItem("userInfo")
              ).memberId;
            }
            this.getTableData(this.searchData.memberId);
          },
        },
        {
          label: "新增",
          type: "primary",
          handle: () => {},
        },
      ],
      searchData: {
        pageNum: this.$route.query.pageNum
          ? parseInt(this.$route.query.pageNum)
          : 1,
        pageSize: 50,
        memberId: [],
        status: "1",
        pageSizeCode: "USER_PAGE_SIZE",
      },
      list: {
        memberList: [],
      },
    };
  },
  created() {
    let memberId = JSON.parse(localStorage.getItem("userInfo")).memberId;
    this.getTableData(memberId);
    this.getDropdownList(memberId);
    this.getPersonalRole(memberId);
  },
  methods: {
    getTableData(memberId) {
      this.$api.arrangement.listArrangement({ memberId }).then((res) => {
        this.tableData = res.data;
        this.total = res.data.length;
      });
    },
    handleEdit(row) {
      // 启动编辑模式
      row.isEditing = true;
    },
    handleEditSave(row) {
      if (row.isEditing) {
        // Save changes
        console.log(
          row.versionOrissueShow.type + ">>>" + row.versionOrissueShow.value
        );
        if (row.id) {
          let data = {
            id: row.id,
            scheduleDate: row.scheduleDate.toString(),
            versionId:
              row.versionOrissueShow.type === "版本"
                ? row.versionOrissueShow.value
                : null,
            issueId:
              row.versionOrissueShow.type === "工单"
                ? row.versionOrissueShow.value
                : null,
            workload: row.workload,
          };
          this.$api.arrangement.edit(data).then(() => {
            this.getTableData(row.memberId);
          });
        } else {
          this.$api.arrangement.add(row).then(() => {
            this.getTableData();
          });
        }
      }
    },
    handleDelete(row) {
      let that = this;
      that.$confirm("是否确认删除？", "消息", {
        callback(action) {
          if (action === "confirm") {
            let data = { id: row.id };
            that.$api.arrangement.deleteArrangemen(data).then(() => {
              that.getTableData(row.memberId);
            });
          }
        },
      });
    },

    getPersonalRole(memberId) {
      this.$api.member.getPersonalRole({ memberId }).then((res) => {
        this.role = res.data;
      });
    },
    handleDateChange(row) {
      const dateString = row.scheduleDate
        .toLocaleDateString("zh-Hans-CN")
        .replace(/\//g, "-");
      const [year, month, day] = dateString
        .split("-")
        .map((s) => s.padStart(2, "0"));
      const formattedDate = `${year}-${month}-${day}`;
      row.scheduleDate = formattedDate;
    },
    getDropdownList(memberId) {
      this.$api.issue
        .getIssueAndVersionDropdownList({ memberId })
        .then((res) => {
          this.options = res.data;
        });
      this.$api.member.getNameByParams().then((res) => {
        this.list.memberList = res.data;
      });
    },
    addNewRow() {
      this.isNewRow = true;
      this.newRow.scheduleDate = new Date().toISOString().substr(0, 10);
    },
  },
};
</script>
<style scoped>
table {
  width: 100%;
  border-collapse: collapse;
}
th,
td {
  border: 1px solid #ccc;
  padding: 8px;
  text-align: left;
}
th {
  background-color: #f2f2f2;
}
</style>
