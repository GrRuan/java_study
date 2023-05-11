<template>
  <div>
    <div>
      <el-dialog
          :title="calculateAndSaveModalTitle"
          :visible.sync="calculateAndSaveModal"
          width="600px"
          class-name="force-fail-modal vertical-center-modal"
      >
        <el-form>
          <el-form-item label="季度" label-width="150px" size="small">
            <el-select v-model="deal.quarterId" clearable filterable>
              <el-option
                  v-for="item in list.quarterIdList"
                  :value="item.value"
                  :key="item.value"
                  :label="item.text"
              ></el-option>
            </el-select>
          </el-form-item>
          <el-form-item
              label="经办人"
              label-width="150px"
              size="small"
              v-show="isReCal"
          >
            <el-select
                v-model="deal.assigneeNameList"
                clearable
                filterable
                default-first-option
            >
              <el-option-group
                  v-for="group in list.nameList"
                  :key="group.text"
                  :label="group.text"
                  placement="top"
              >
                <el-option
                    v-for="item in group.options"
                    :key="item.value"
                    :label="item.text"
                    :value="item.value"
                >
                </el-option>
              </el-option-group>
            </el-select>
          </el-form-item>
        </el-form>
        <div slot="footer">
          <el-button type="primary" class="subBtn" @click="submit"
          >提交
          </el-button
          >
          <el-button type="primary" class="subBtn" @click="cancel"
          >取消
          </el-button
          >
        </div>
      </el-dialog>
      <HighQuery ref="highQuery" :queryDatas="queryDatas" @on-highQuery="highQuery"></HighQuery>
    </div>
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
          :cell-style="setCellStyle"
          @on-cell-dblclick="cellChange"
          @on-select-all="onSelectAll"
          @on-select="select"
          @on-selection-change="selectChange"
          @on-change="pageChange"
          @on-sort-change="sortChange"
          @on-page-size-change="pageSizeChange"
      ></r-table>
    </el-main>
  </div>
</template>

<script>
import HighQuery from '../highQuery/highQuery.vue' // 高级查询
export default {
  name: "scoreList",
  components: {HighQuery},
  data() {
    return {
      queryDatas: [], // 高级查询字段
      searchForm: [
        {
          type: "Select",
          label: "团队",
          prop: "department",
          placeholder: "请选择",
          clearable: true,
          multiple: true,
          change: value => {
            this.searchData.department = value;
          }
        },
        {
          type: "SelectGroup",
          label: "姓名",
          prop: "name",
          placeholder: "请选择",
          clearable: true,
          multiple: true,
          change: value => {
            this.searchData.name = value;
          }
        },
        {
          type: "Select",
          label: "角色",
          prop: "roleCode",
          placeholder: "请选择",
          clearable: true,
          multiple: true,
          change: value => {
            this.searchData.roleCode = value;
          }
        },
        {
          type: "Select",
          label: "季度",
          prop: "quarterId",
          placeholder: "请选择",
          clearable: true,
          multiple: true,
          collapse: true,
          change: value => {
            this.searchData.quarterId = value;

          },

        },
      ],
      searchHandle: [
        {
          label: "查询",
          type: "primary",
          handle: () => {
            this.getTableData();
          }
        },


      ],
      tableHandles: [
        {
          label: "高级查询",
          type: "primary",
          handle: () => {
            this.HighQuery();
          }
        },
        {
          label: "绩效计算",
          type: "primary",
          btn: "calculateAndSave",
          handle: () => {
            this.calculateAndSaveModal = true;
          }
        },
        {
          label: "重新汇总分数并排名",
          type: "primary",
          btn: "reCalculateSummary",
          handle: () => {
            this.calculateAndSaveModal = true;
            this.isReCal = true
          }
        }
      ],
      deal: {
        quarterId: "",
        assigneeNameList: []
      },
      tableData: [],
      total: 0,
      sortName: "",
      sortType: "",
      isReCal: false,
      calculateAndSaveModalTitle: "绩效分计算",
      calculateAndSaveModal: false, // 绩效计算弹窗
      columns: [
        {key: "realName", label: "用户名", prop: "realName", sort: 'custom'},
        {key: "roleCodeShow", label: "角色", prop: "roleCodeShow", sort: true},
        {key: "quarterShow", label: "季度", prop: "quarterShow", sort: 'custom', sortName: "quarter"},
        {key: "spendTime", label: "总工时", prop: "spendTime", sort: 'custom'},
        {key: "originScore", label: "原始总绩效分", prop: "originScore", sort: 'custom'},
        {key: "score", label: "最终总绩效分", prop: "score", sort: 'custom'},
        {key: "ranking", label: "总排名", prop: "ranking", sort: 'custom'},
        {key: "position", label: "职级", prop: "position", sort: true, width: "80"},
        {key: "calculatedScore", label: "权重分", prop: "calculatedScore", sort: 'custom'},
        {key: "issueCount", label: "工单数量", prop: "issueCount", sort: 'custom'},
        {key: "specialIssueCount", label: "特殊工单数量", prop: "specialIssueCount", sort: 'custom'},
        {key: "specialIssueScore", label: "特殊工单积分", prop: "specialIssueScore", sort: 'custom'},
        {
          key: "specialIssueCountRatShow",
          label: "特殊工单数量占比",
          prop: "specialIssueCountRatShow",
          sort: 'custom',
          sortName: "specialIssueCountRat"
        },
        {
          key: "specialIssueScoreRatShow",
          label: "特殊工单积分占比",
          prop: "specialIssueScoreRatShow",
          sort: 'custom',
          sortName: "specialIssueScoreRat"
        },
        {key: "teamRanking", label: "团队内排名", prop: "teamRanking", sort: 'custom'},
      ],
      selection: [],
      searchData: {
        pageNum: this.$route.query.pageNum
            ? parseInt(this.$route.query.pageNum)
            : 1,
        pageSize: 50,
        department: [],
        name: [],
        quarterId: [],
        roleCode:[],
        pageSizeCode: 'USER_PAGE_SIZE',
      },
      list: {
        departmentList: [],
        nameList: [],
        quarterIdList: [],
        roleCodeList: []
      }
    };
  },
  created() {
    this.getDropdownList();
    this.getTableData();
    this.getNameList();
  },
  watch: {
    "searchData.department"(val) {
      this.getNameList(val);
    },
    isReCal(val) {
      if (val) {
        this.calculateAndSaveModalTitle = "重新计算成员绩效分"
      } else {
        this.calculateAndSaveModalTitle = "绩效分计算"
      }
    }
  },
  activated() {
    this.getTableData();
  },

  methods: {
    getTableData: async function (params) {
      let that = this;
      await that.$api.config.getPageSize({code: this.searchData.pageSizeCode,}).then(res => {
        this.searchData.pageSize = parseInt(res.data);
      });
      let data = {
        pageNum: this.searchData.pageNum,
        pageSize: this.searchData.pageSize,
        sortName: this.sortName,
        sortType: this.sortType,
        departmentList: that.searchData.department,
        nameList: that.searchData.name,
        quarterIdList: that.searchData.quarterId,
        roleCodeList: that.searchData.roleCode,
        ...params
      };
      await that.$api.score.listScore(data).then(res => {
        that.tableData = res.rows;
        that.total = res.total;
        if (params) {
          this.$refs.highQuery.highQueryModal = false
        }
      });

    },

    getDropdownList() {
      //获取组下拉信息
      this.$api.department.getDepartmentDropDownList().then((res) => {
        this.list.departmentList = res.data;
      });

      this.$api.common.getProjectList().then(res => {
        this.list.projectList = res.data;
      });

      this.$api.common.getQuarterList().then(res => {
        this.list.quarterIdList = res.data;    
        //季度信息下拉列表已经按id降序排列了，获取第一个
        let ids = [];
        ids.push(res.data[0].value);
        this.searchData.quarterId = ids;
      });
      //获取员工所属角色下拉
      this.$api.common.getDropdownListByType({type: '8'}).then(res => {
        this.list.roleCodeList = res.data;
      });

    },
    getVersionList(val) {
      let projectList = val !== undefined ? val.join(",") : "";
      this.$api.common
          .getVersionList({projectList: projectList})
          .then(res => {
            this.list.versionList = res.data;
          });
    },
    submit() {
      let that = this
      let data = {
        quarterId: this.deal.quarterId,
        memberNameList: this.deal.assigneeNameList
      };
      let api = this.isReCal ? this.$api.score.reCalculateSummary(data) : this.$api.score.calculateAndSave(data)
      api.then(res => {
        if (res.code === "0") {
          that.$alert(res.info);
          that.calculateAndSaveModal = false;
        }
      });
      this.isReCal = false
    },
    cancel() {
      this.calculateAndSaveModal = false;
      this.deal.quarterId = "",
          this.isReCal = false
    },
    getNameList(val) {
      let list = val !== undefined ? val.join(",") : "";
      this.$api.common.getAssigneeList({list: list}).then(res => {
        this.list.nameList = res.data;
      });
    },

    getShowField() {
      this.$refs.highQuery.getShowFieldByUrl('/scoreList')
    },

    // 高级查询弹窗
    HighQuery() {
      this.getShowField()
      this.$refs.highQuery.highQueryModal = true
    },

    // 高级查询
    highQuery(params) {
      this.searchData.pageNum = 1
      this.getTableData(params)
    },

    pageChange(page) {
      this.searchData.pageNum = page;
      this.getTableData(page);
    },
    pageSizeChange(pageSize) {
      if (this.searchData.pageSize !== pageSize) {
        this.searchData.pageSize = pageSize;
        this.searchData.pageNum = 1;
        this.updatePagingStrategy(pageSize)
      }
    },
    async updatePagingStrategy(pageSize){
      await this.$api.config.setPageSize({code: this.searchData.pageSizeCode, value: pageSize});
      await this.getTableData()
    },
    select: function (selection) {
      this.selection = selection;
    },
    selectChange: function (selection) {
      this.selection = selection;
    },
    onSelectAll: function (selection) {
      this.selection = selection;
    },
    // 排序时生效
    sortChange(sortName, sortType) {
      this.sortName = sortName;
      this.sortType = sortType;
      this.searchData.pageNum = 1;
      this.getTableData();
    },

    cellChange(row, column) {
      if (column.property === "realName") {
        this.$emit("routerHandle", "/issue/issueList", {
          title: "工单明细",
          query: {
            name: row.name,
            quarterId: row.quarterId.toString()
          }
        });

        // this.$root.$children[0].activeInfo
      }
    },
    setCellStyle({row, column}) {
      if (row && column.property === 'realName') {
        return {
          color: '#00BFFF'
        }
      }
      return {}
    }
  }
};
</script>

<style scoped></style>
