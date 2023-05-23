<template>
  <div>
    <Query
      size="mini"
      labelWidth="80px"
      :itemWidth="7"
      :searchData="searchData"
      :searchForm="searchForm"
      :searchHandle="searchHandle"
      :list="list"
    ></Query>
    <el-main class="table-main">
      <r-table
        ref="myTable"
        :tableData="tableData"
        :columns="columns"
        :uniqueID="'id'"
        :current="searchData.pageNum"
        :pageSize="searchData.pageSize"
        :total="total"
        :multiSelect="true"
        :isHandle="true"
        :hasChildren="true"
        :tableHandles="tableHandles"
        :cell-style="setCellStyle"
        :row-class-name="currentRowClassName"
        @on-select-all="onSelectAll"
        @on-cell-dblclick="cellChange"
        @on-select="select"
        @on-change="pageChange"
        @on-page-size-change="pageSizeChange"
        @on-sort-change="sortChange"
        @on-row-click="rowClick"
      ></r-table>
    </el-main>
  </div>
</template>

<script>
export default {
  name: "department",
  data() {
    return {
      searchForm: [],
      searchHandle: [],
      tableHandles: [
        {
          label: "新增",
          type: "primary",
          handle: () => {
            this.$router.push({
              path: "/department/add",
              query: {
                state: "add",
              },
            });
          },
        },
        {
          label: "编辑",
          type: "primary",
          handle: () => {
            if (this.selection.length === 0) {
              this.$message.warning("请至少选中一项记录");
              return;
            }
            if (this.selection.length !== 1) {
              this.$message.warning("只能选中一项记录进行编辑");
              return;
            }
            this.$router.push({
              path: "/department/add",
              query: {
                state: "edit",
                data: this.selection[0],
              },
            });
          },
        },
        {
          label: "禁用",
          type: "primary",
          handle: () => {
            let that = this;
            if (that.selection.length === 0) {
              that.$message.warning("请至少选中一项记录");
              return;
            }
            that.$confirm("是否确认禁用", "消息", {
              callback(action) {
                if (action === "confirm") {
                  that.$api.department
                    .batchDelete({
                      ids: that.selection.map((id) => id).join(","),
                    })
                    .then((res) => {
                      that.$message.success(res.info);
                      that.getTableData();
                    });
                }
              },
            });
          },
        },
      ],
      tableData: [],
      total: 0,
      columns: [
        {
          key: "name",
          label: "团队名称",
          prop: "name",
          sort: true,
          render: (h, { row }) => {
            // 判断当前行的可编辑状态
            if (row.editable) {
              return h("el-input", {
                props: {
                  value: row.teamName,
                },
                on: {
                  input: (value) => {
                    // 双向绑定
                    row.teamName = value;
                  },
                },
              });
            } else {
              return h("span", row.teamName);
            }
          },
        },
        {
          key: "dingtalkDeptId",
          label: "钉钉团队编号",
          prop: "dingtalkDeptId",
          sort: true,
          render: (h, { row }) => {
            // 判断当前行的可编辑状态
            if (row.editable) {
              return h("el-input", {
                props: {
                  value: row.dingTalkTeamNumber,
                },
                on: {
                  input: (value) => {
                    // 双向绑定
                    row.dingTalkTeamNumber = value;
                  },
                },
              });
            } else {
              return h("span", row.dingTalkTeamNumber);
            }
          },
        },
        {
          label: "操作",
          width: 150,
          render: (h, { row }) => {
            // 根据当前行的编辑状态显示不同的按钮
            if (row.editable) {
              return h("div", [
                h(
                  "el-button",
                  {
                    props: {
                      type: "primary",
                      size: "mini",
                    },
                    on: {
                      click: () => {
                        // 点击保存按钮后，将当前行的编辑状态设置为 false，并将修改后的数据提交到后台
                        row.editable = false;
                        console.log("提交数据：", row);
                        // TODO: 将修改后的数据提交到后台
                      },
                    },
                  },
                  "保存"
                ),
                h(
                  "el-button",
                  {
                    props: {
                      type: "text",
                      size: "mini",
                    },
                    on: {
                      click: () => {
                        // 点击取消按钮后，将当前行的编辑状态设置为 false，并取消修改
                        row.editable = false;
                        console.log("取消修改：", row);
                        // TODO: 取消修改
                      },
                    },
                  },
                  "取消"
                ),
              ]);
            } else {
              return h("div", [
                h(
                  "el-button",
                  {
                    props: {
                      type: "primary",
                      size: "mini",
                    },
                    on: {
                      click: () => {
                        // 点击编辑按钮后，将当前行的编辑状态设置为 true
                        row.editable = true;
                      },
                    },
                  },
                  "编辑"
                ),
              ]);
            }
          },
        },
      ],
      selection: [],
      searchData: {
        //暂无查询条件
      },
      list: {
        //暂无下拉列表查询条件
      },
    };
  },
  created() {
    this.getTableData();
  },
  watch: {},
  activated() {
    this.getTableData();
  },

  methods: {
    getTableData: function () {
      let that = this;
      that.$api.department.getDepartmentList().then((res) => {
        that.tableData = res.data;
        that.total = res.data.length;
      });
    },
    //表格选中项改变时触发，data为当前最新的选中项信息
    selectChange(data) {
      this.selection = data.map((department) => department.id);
    },
    rowClick(row) {
      this.tableData.forEach((item) => {
        item.editable = false;
      });
      row.editable = true;
    },
    setCellStyle({ row }) {
      return row.editable ? { backgroundColor: "#f5f7fa" } : {};
    },
  },
};
</script>

<style scoped>
</style>