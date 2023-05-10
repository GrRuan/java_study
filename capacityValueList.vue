<template>
  <div>
    <div>
      <el-dialog
          :title="jiraModalTitle"
          :visible.sync="jiraModal"
          width="700px"
          class-name="force-fail-modal vertical-center-modal"
      >
        <div>
          <el-form
              ref="form"
              :inline="true"
              size="mini"
              label-position="left"
              :model="form" style="height: 300px; overflow: auto;"
          >
            <el-row v-for="(item, index) in form.dynamicItem" :key="index">
              <el-col span=12>
                <el-form-item
                    label="所属角色"
                    :key="index"
                    :prop="'dynamicItem.' + index + '.roleCode'"
                    :rules="{
      required: true, message: '所属角色不能为空', trigger: 'blur'
    }"
                >
                  <el-select
                      v-model="item.roleCode"
                      clearable
                      placeholder="请选择">
                    <el-option
                        v-for="option in list.roleCodeList"
                        :key="option.value"
                        :label="option.text"
                        :value="option.value">
                    </el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col span=12>
              <el-form-item
                  label="职级"
                  :key="item.positionList"
                  :prop="'dynamicItem.' + index + '.positionList'">
                <el-select
                    v-model="item.positionList"
                    clearable
                    placeholder="请选择" multiple>
                  <el-option
                      v-for="option in list.positionList"
                      :key="option.value"
                      :label="option.text"
                      :value="option.value">
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
              <el-col span=12>
                <el-form-item
                    label="重要性"
                    :key="index"
                    :prop="'dynamicItem.' + index + '.capacityImportanceCode'"
                    :rules="{
      required: true, message: '重要性不能为空', trigger: 'blur'
    }"
                >
                  <el-col>
                    <el-select v-model="item.capacityImportanceCode" clearable placeholder="请选择">
                      <el-option
                          v-for="option in list.capacityImportanceCodeList"
                          :key="option.value"
                          :label="option.text"
                          :value="option.value">
                      </el-option>
                    </el-select>
                  </el-col>
                </el-form-item>
                <el-form-item>
                  <i class="el-icon-delete" @click="deleteItem(item, index)"></i>
                </el-form-item>
              </el-col>
            </el-row>
          </el-form>
          <el-form>
            <el-row>
              <el-col span=16 style="border: 1px solid transparent">
              </el-col>
              <el-col span=8>
                <el-form-item>
                  <el-button @click="addItem()" type="primary">增加角色和重要性</el-button>
                </el-form-item>
              </el-col>
            </el-row>
          </el-form>
          <div slot="footer">
            <el-button type="primary" class="subBtn" @click="submit()"
            >提交
            </el-button
            >
            <el-button type="primary" class="subBtn" @click="cancel"
            >取消
            </el-button
            >
          </div>
        </div>
      </el-dialog>
    </div>
    <Query
        size="mini"
        labelWidth="100px"
        :itemWidth="7"
        :searchData="searchData"
        :searchForm="searchForm"
        :searchHandle="searchHandle"
        :list="list"></Query>
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
          :hasChildren=true
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
  name: "capacityValueList",
  data() {
    return {
      form: {
        dynamicItem: [
          {
            roleCode: null,
            positionList: null,
            capacityImportanceCode: null,
          }
        ]
      },
      labelPosition: 'right',
      rules: {
        capacityImportanceCode: [{required: true, message: '能力明细项重要性是必填项', trigger: 'blur'}],
        roleCode: [{
          required: true, message: '能力明细项角色是必填项'
          , trigger: 'blur'
        }]
      },
      searchForm: [
      {
          type: "Select",
          label: "能力项分类",
          prop: "capacityTypeId",
          placeholder: "请选择",
          clearable: true,
          multiple: true,
          change: value => {
            console.log("选中的值：", value);
            this.searchData.capacityTypeId = value;
            this.getCapacityDropdownList();
          },
        },
        {
          type: "Select",
          label: "能力项名称",
          prop: "capacityId",
          placeholder: "请选择",
          clearable: true,
          multiple: true,
          change: value => {
            this.searchData.capacityId = value;
          },
        },
        {
          type: "Select",
          label: "能力项所属角色",
          prop: "roleCode",
          placeholder: "请选择",
          clearable: true,
          multiple: true,
          change: value => {
            this.searchData.roleCode = value;
          },
        },
        {
          type: "Select",
          label: "能力项重要性",
          prop: "capacityImportanceCode",
          placeholder: "请选择",
          clearable: true,
          multiple: true,
          change: value => {
            this.searchData.capacityImportanceCode = value;
          },
        },
      ],
      searchHandle: [
        {
          label: "查询",
          type: "primary",
          handle: () => {
            //清空selection
            this.selection = [];
            this.getTableData();
            console.log(this.tableData[0])
            console.log(this.tableData[0].children[0])
            let columnName = 'roleAndImportanceCodeShow';
            for (let i = 0; i < this.tableData.length; i++) {
              let row = this.tableData[i];
              for (let j = 0; j < row.children.length; j++) {
                let roleAndImportance = row.children[j][columnName];        
                if(roleAndImportance!==null){
                  row.children[j][columnName]=roleAndImportance.replace(/\n/g,"<br>");
                  console.log(row.children[j][columnName])
                }
            }
            this.tableData = JSON.parse(JSON.stringify(this.tableData));
          }
          this.tableData = JSON.parse(JSON.stringify(this.tableData));
        },
      }
      ],
      tableHandles: [
        {
          label: "新增",
          type: "primary",
          size: "mini",
          page: "capacityValue",
          btn: "Add",
          handle: () => {
            this.$router.push({
              path: "/capacityValue/add",
              query: {
                state: "add",
                capacityId:this.searchData.capacityId,
                capacityTypeId:this.searchData.capacityTypeId,
                roleCode:this.searchData.roleCode,
                position:this.searchData.position,
                capacityImportanceCode:this.searchData.capacityImportanceCode
            }
            });
          }
        },
        {
          label: "编辑",
          type: "primary",
          size: "mini",
          page: "capacityValue",
          btn: "Edit",
          handle: () => {
            if (this.selection.length !== 1) {
              this.$message.warning("请选择一条操作记录！");
              return;
            }
            this.$router.push({
              path: "/capacityValue/add",
              query: {
                data: this.selection[0].id,
                state: "edit",
                pageNum: this.searchData.pageNum,
                capacityId:this.searchData.capacityId,
                capacityTypeId:this.searchData.capacityTypeId,
                roleCode:this.searchData.roleCode,
                position:this.searchData.position,
                capacityImportanceCode:this.searchData.capacityImportanceCode
              }
            });
          }
        },
        {
          label: "删除",
          type: "primary",
          size: "mini",
          page: "capacityValue",
          btn: "delete",
          handle: () => {
            let that = this
            console.log(that.selection)
            if (that.selection.length === 0) {
              that.$message.warning("请至少选择一条操作记录！");
              return;
            }
            let data = {ids: that.selection.map(value => value.id).join(',')}

            that.$confirm("是否确认删除？", "消息", {
              callback(action) {
                if (action === "confirm") {
                  that.$api.capacity.delCapacityValueById(data)
                      .then(res => {
                        that.$message.success(res.info);
                        that.getTableData();
                      });
                }
              }
            });
          }
        },
        {
          label: "分配角色和重要性",
          type: "primary",
          btn: "getJira",
          page: "capacityValueList",
          handle: () => {
            if (this.selection.length < 1) {
              this.$message.warning("请选择一条或者多条操作记录！");
              return;
            } else {
              this.jiraModal = true;
            }
          }
        },
      ],
      tableData: [],
      jiraModalTitle: "设置员工角色和重要性",
      jiraModal: false,
      total: 0,
      appDocData: '',
      sortName: '',
      sortType: '',
      columns: [
        {key: "capacityShow", label: "能力项名称", prop: "capacityShow", sort: "custom", sortName: "capacityId"},
        {key: "capacityTypeName", label: "能力项分类", prop: "capacityTypeName", sort: "custom", sortName: "capacityTypeId"},
        {key: "value", label: "能力项分值 ", prop: "value", sort: "custom"},
        {key: "info", label: "详细文字描述", prop: "info", sort: "custom"},
        {key: "weight", label: "权重", prop: "weight", sort: "custom"},
        {key: "weightInfo", label: "权重详细文字描述", prop: "weightInfo", sort: "custom"},
        {key: "roleAndImportanceCodeShow", label: "角色职级：重要性", prop: "roleAndImportanceCodeShow", sort: "custom"},
        {key: "learnTime", label: "学习时间（小时）", prop: "learnTime", sort: "custom"},
        {key: "practiceTime", label: "实践时间（小时）", prop: "practiceTime", sort: "custom"},
        {key: "learnCycle", label: "学习周期（天）", prop: "learnCycle", sort: "custom"},
        {key: "practiceCycle", label: "实践周期（天）", prop: "practiceCycle", sort: "custom"},
        {key: "cost", label: "费用", prop: "cost", sort: "custom"},
      ],
      selection: [],
      fatherSelection: [],
      searchData: {
        pageNum: this.$route.query.pageNum
            ? parseInt(this.$route.query.pageNum)
            : 1,
        pageSize: 50,
        capacityId: this.$route.query.capacityId,
        capacityTypeId: this.$route.query.capacityTypeId,
        roleCode: this.$route.query.roleCode,
        position: this.$route.query.position,
        capacityImportanceCode: this.$route.query.capacityImportanceCode,
        info: '',
        pageSizeCode: 'USER_PAGE_SIZE',
      },
      list: {
        capacityTypeIdList: [],
        capacityIdList: [],
        roleCodeList: [],
        positionList: [],
        capacityImportanceCodeList: [],
      },
    }
  },
  mounted() {
    if (this.$route.query) {
      if (this.$route.query.capacityTypeId) {
        this.searchData.capacityTypeId = (this.$route.query.capacityTypeId);
      }
    }
    if (this.$route.query) {
      if (this.$route.query.capacityId) {
        this.searchData.capacityId = (this.$route.query.capacityId);
      }
    }
    this.getTableData();
    this.getCapacityDropdownList();
    this.getCapacityTypeDropdownList();
    this.getRoleCodeDropdownList();
    this.getCapacityPositionList();
    this.getCapacityDetailImportanceDropDownList();
  },

  methods: {
    //新增加一栏
    addItem: function () {
      this.form.dynamicItem.push({
        roleCode: null,
        positionList: null,
        capacityImportanceCode: null
      })
    },
    //减少一栏
    deleteItem: function (item, index) {
      if (this.form.dynamicItem.length === 1) {
        this.$message.warning("能力明细项至少有一组角色和重要性");
        return;
      }
      this.form.dynamicItem.splice(index, 1)

    },
    //获取能力项所属角色下拉
    getRoleCodeDropdownList: function () {
      this.$api.common.getDropdownListByType({type: '8'}).then(res => {
        this.list.roleCodeList = res.data;
      });
    },
    //获取职级下拉
    getCapacityPositionList: function () {
      this.$api.common.getDropdownListByType({type: '4'}).then(res => {
        this.list.positionList = res.data;
      });
    },
    //获取能力项重要性下拉
    getCapacityDetailImportanceDropDownList: function () {
      this.$api.common.getDropdownListByType({type: '9'}).then(res => {
        this.list.capacityImportanceCodeList = res.data;
      });
    },
    submit() {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          let data = {
            roleAndImportance: this.form.dynamicItem,
            capacityValueIdList: this.selection.map(value => value.id),
          };
          //调用方法
          if (data.roleAndImportance !== null && data.roleAndImportance.length > 0) {
            if (data.roleAndImportance.length === 1) {
              if (data.roleAndImportance[0].roleCode === null || data.roleAndImportance[0].capacityImportanceCode === null) {
                data.roleAndImportance = null;
              }
            }
          }
          let that = this;
          this.$api.capacity.batchAddCapacityRoleAndImpo(JSON.stringify(data)).then(res => {
            that.jiraModal = false;
            that.getTableData();
            that.$message.success(res.info);
            ;
          });
          //清空上一次选择的数据
          this.selection = [];
          this.form.dynamicItem = [
            {
              roleCode: null,
              positionList: null,
              capacityImportanceCode: null,
            }
          ];

        } else {
          return false;
        }
      });
    },
    cancel() {
      this.jiraModal = false;
      for (var i = 0; i < this.form.dynamicItem.length; i++) {
        this.form.dynamicItem[i].roleCode = '';
        this.form.dynamicItem[i].position = '';
        this.form.dynamicItem[i].capacityImportanceCode = '';
      }
    },
    clearSearchData() {
      this.searchData.capacityTypeId = [];
      this.searchData.capacityId = [];
      this.searchData.info = '';
      this.searchData.weight = '';
      this.searchData.weightInfo = '';
    },

    getTableData: function (page) {
      let that = this
      let data = {
        pageNum: page ? page : 1,
        pageSize: this.searchData.pageSize,
        sortName: this.sortName,
        sortType: this.sortType,
        capacityTypeIdList: this.searchData.capacityTypeId,
        capacityIdList: this.searchData.capacityId,
        roleCodeList: this.searchData.roleCode,
        position: this.searchData.position,
        positionList: this.searchData.position,
        capacityImportanceCodeList: this.searchData.capacityImportanceCode,
        value: this.searchData.info,
        infoUrl: this.searchData.infoUrl,
        weight: this.searchData.weight,
        weightInfo: this.searchData.weightInfo
      }
      that.$api.capacity.getCapacityValueList(data).then(res => {
        that.tableData = res.data;
        // that.tableData = res.rows;
        // that.total = res.total;
      });
    },
    getCapacityDropdownList() {
    // const capacityTypeIdList = this.list.capacityTypeIdList.map(({ value }) => value)
    // const capacityTypeIdList = Array.from(this.searchData.capacityTypeId);
    // const data = capacityTypeIdList;
    let data = {capacityTypeIdList: this.searchData.capacityTypeId.map(value => value).join(',')}
    this.$api.capacity.getCapacityDropdownList(data).then(res => {
        const capacityIdList = res.data.map(item => ({
        text: item.text,
        value: item.value
      }));
      this.list.capacityIdList = capacityIdList;
    });
    },
    getCapacityTypeDropdownList() {
      this.$api.capacity.getCapacityTypeDropdownList().then(res => {
      const capacityTypeIdList = res.data.map(item => ({
        text: item.text,
        value: item.value
      }));
      this.list.capacityTypeIdList = capacityTypeIdList;
    });
    },

    // 单击行
    rowClick(row) {
      if (
          this.fatherSelection.some(el => {
            return row.id === el.id;
          })
      ) {
        // 点击已经选中的父节点行，将该父节点的所有子节点从selection移除
        row.children.forEach(x => {
          this.selection.splice(this.selection.indexOf(x), 1);
        });
        this.setChildren(row.children, false);
        // 从fatherSelection中移除点击的节点
        this.fatherSelection.splice(this.fatherSelection.indexOf(row), 1);
      } else {
        if (
            this.selection.some(el => {
              return row.id === el.id;
            })
        ) {
          // 点击已经选中的子节点行，将该子节点从selection中移除，并将该节点的选中状态设置为false
          this.selection.splice(this.selection.indexOf(row), 1);
          row.selectionFlag = false;
        } else {
          this.selected(row);
        }
      }
    },

    select(selection, row) {
      if (selection.some(el => {
        return row.id === el.id
      })) {
        this.selected(row)
      } else {
        this.canceled(row, selection)
      }
    },
    // 选择全部
    onSelectAll(selection) {
      this.selection = []
      const isSelect = selection.some(el => {
        const tableDataIds = this.tableData.map(j => j.id)
        return tableDataIds.includes(el.id)
      })
      // tableDate第一层只要有不在selection里面就是全不选
      const isCancel = !this.tableData.every(el => {
        const selectIds = selection.map(j => j.id)
        return selectIds.includes(el.id)
      })
      if (isSelect) {
        selection.map(el => {
          if (el.children) {
            //
            el.children.forEach(x => {
              this.selection.push(x)
            })
            // 勾选子节点
            this.setChildren(el.children, true)
          }
        })
      }
      if (isCancel) {
        this.tableData.map(el => {
          if (el.children) {
            // 因为一开始就置空selection，所以取消选中无需处理 this.selection
            // el.children.forEach(x=>{
            //     this.selection.splice(this.selection.indexOf(x), 1)
            // })
            // 取消勾选子节点
            this.setChildren(el.children, false)
          }
        })
      }
      this.$emit('handleSelect', this.tableData)
    },

    // 选中：包括处理选中数据依据打勾
    selected(row) {
      if (row.children) {
        // 若选中的是父节点，则需要将所有子节点置为选中状态，且将其push进selection中
        row.children.forEach(x => {
          if (!this.selection.includes(x)) {
            this.selection.push(x)
          }
        })
        // 勾选子节点
        this.setChildren(row.children, true)
        // 若选择的节点存在子节点，则将该节点push到fatherSelection
        this.fatherSelection.push(row);
      } else {
        this.selection.push(row)
        this.setSelf(row, true)

      }
    },

    // 取消选中：包括处理选中数据依据打勾
    // 若选中的是父节点，则需要将所有子节点置为取消选中状态，且将其从selection中splice
    canceled(row, selection) {
      if (row.children) {
        row.children.forEach(x => {
          this.selection.splice(this.selection.indexOf(x), 1)
        })
        this.setChildren(row.children, false)

      } else {
        this.selection.splice(selection, 1)
        this.setSelf(row, false)

      }
      // if (row.children) {
      //     this.setChildren(row.children, false)
      // }else {
      //     this.setSelf(row, false)
      // }
    },

    //选中父节点时，设置子节点选中标记，打勾、取消打勾
    setChildren(children, type) {
      children.map(j => {
        this.toggleSelection(j, type)
        j.selectionFlag = type
        if (j.children) {
          this.setChildren(j.children, type)
        }
      })
    },

    //点击子节点时，设置选中标记，打勾、取消打勾
    setSelf(row, type) {
      row.selectionFlag = type
      this.$refs.myTable.toggleRowSelection(row, type)
    },

    // 行打勾
    toggleSelection(row, select) {
      if (row) {
        this.$nextTick(() => {
          this.$refs.myTable && this.$refs.myTable.toggleRowSelection(row, select)
        })
      }
    },

    // 父节点checkBox样式(取消选中，全选，半选)
    currentRowClassName(rowInfo) {
      let row = rowInfo.row
      // 如果为父节点
      if (row.children) {
        // 判断父节点下属子节点是否为全部选中状态
        const allSelect = row.children.every(el => {
          return el.selectionFlag === true
        })
        // 判断父节点下属子节点是否为全部取消选中状态
        const allCancel = row.children.every(el => {
          return el.selectionFlag !== true
        })
        // 设置为半选样式
        if (!allSelect && !allCancel) {
          return 'el-checkbox__input is-indeterminate';
        }
        // 恢复取消选中样式
        if (allCancel) {
          // 取消勾选父节点
          this.setSelf(row, false)
          return 'el-checkbox__input';
        }
        // 设置为全选样式
        if (allSelect) {
          return 'el-checkbox__input is-checked';
        }
      }
    },

    pageChange(page) {
      this.getTableData(page);
    },
    pageSizeChange(pageSize) {
      if (this.searchData.pageSize !== pageSize) {
        this.searchData.pageSize = pageSize;
        this.searchData.pageNum = 1;
        this.getTableData();
      }
    },
    // 排序时生效
    sortChange(sortName, sortType) {
      this.sortName = sortName
      this.sortType = sortType
      this.getTableData()
    },

    setCellStyle({row, column}) {
      let cellStyle
      // 父节点样式调整
      if (row && column.property === 'capacityShow') {
        cellStyle = 'color:blue;white-space: nowrap;font-size: 14px;  font-weight: 600;'
      }
      if (row && column.property === 'roleAndImportanceCodeShow') {
        cellStyle = 'white-space: pre-wrap !important ;font-size: 14px;  font-weight: 600;'
      }
      if (row && column.property === 'info') {
        return {
          color: '#00BFFF',
          cursor: 'pointer',
        }
      }
      return cellStyle
    },
    cellChange(row, column) {
      if (column.property === "info") {
        if (row.infoUrl !== null && row.infoUrl !== "") {
          window.open(row.infoUrl)
        } else {
          this.getUrl(row);
        }
      }
    },
    getUrl(row) {
      this.$api.capacity.getCapacityValueUrl(row).then(res => {
        if (res.data !== null && res.data !== "") {
          window.open(res.data)
        } else {
          this.$message("未找到相关文章");
        }
      });
    },
  },
  // beforeRouteEnter(to, from, next) {
  //     if (from.name === "capacity") {
  //         next(vm => {
  //             vm.clearSearchData();
  //             if (vm.$route.query.capacityId) {
  //                 vm.searchData.capacityId.push(vm.$route.query.capacityId);
  //             }
  //             vm.getTableData();
  //         });
  //     } else {
  //         next();
  //     }
  // }
}
</script>

<style type="less">
</style>
