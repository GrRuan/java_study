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
        :tableHandles="tableHandles"
        @on-selection-change="selectChange"
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
                state: "add"
              }
            });
          }
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
              }
            });
          }
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
                      ids: that.selection.map(id => id).join(",")
                    })
                    .then(res => {
                      that.$message.success(res.info);
                      that.getTableData();
                    });
                }
              }
            });
          }
        }
      ],
      tableData: [],
      total: 0,
      columns: [
        { key: "name", label: "团队名称", prop: "name", sort: true },
        {
          key: "dingtalkDeptId",
          label: "钉钉团队编号",
          prop: "dingtalkDeptId",
          sort: true
        }
      ],
      selection: [],
      searchData: {
        //暂无查询条件
      },
      list: {
        //暂无下拉列表查询条件
      }
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
    getTableData: function() {
      let that = this;
      that.$api.department.getDepartmentList().then(res => {
        that.tableData = res.data;
        that.total = res.data.length;
      });
    },
    //表格选中项改变时触发，data为当前最新的选中项信息
    selectChange(data) {
      this.selection = data.map(department => department.id);
    }
  }
};
</script>

<style scoped>
</style>