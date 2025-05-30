<template> 
  <div class="app-container">
    <el-card class="filter-container" shadow="never">
      <div>
        <i class="el-icon-search"></i>
        <span>Filter Search</span>
        <el-button
          style="float: right"
          @click="handleSearchList()"
          type="primary"
          size="small">
          Search
        </el-button>
        <el-button
          style="float: right;margin-right: 15px"
          @click="handleResetSearch()"
          size="small">
          Reset
        </el-button>
      </div>
      <div style="margin-top: 15px">
        <el-form :inline="true" :model="listQuery" size="small" label-width="140px">
          <el-form-item label="Search:">
            <el-input style="width: 203px" v-model="listQuery.keyword" placeholder="Product Name"></el-input>
          </el-form-item>
          <el-form-item label="Product Code:">
            <el-input style="width: 203px" v-model="listQuery.productSn" placeholder="Product Code"></el-input>
          </el-form-item>
          <el-form-item label="Category:">
            <el-cascader
              clearable
              v-model="selectProductCateValue"
              :options="productCateOptions">
            </el-cascader>
          </el-form-item>
          <el-form-item label="Brand:">
            <el-select v-model="listQuery.brandId" placeholder="Select Brand" clearable>
              <el-option
                v-for="item in brandOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="Publish Status:">
            <el-select v-model="listQuery.publishStatus" placeholder="All" clearable>
              <el-option
                v-for="item in publishStatusOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="Verification Status:">
            <el-select v-model="listQuery.verifyStatus" placeholder="All" clearable>
              <el-option
                v-for="item in verifyStatusOptions"
                :key="item.value"
                :label="item.label"
                :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </div>
    </el-card>

    <el-card class="operate-container" shadow="never">
      <i class="el-icon-tickets"></i>
      <span>Product List</span>
      <el-button
        class="btn-add"
        @click="handleAddProduct()"
        size="mini">
        Add
      </el-button>
    </el-card>

    <div class="table-container">
      <el-table
        ref="productTable"
        :data="list"
        style="width: 100%"
        @selection-change="handleSelectionChange"
        v-loading="listLoading"
        border>
        <el-table-column type="selection" width="60" align="center"></el-table-column>
        <el-table-column label="ID" width="100" align="center">
          <template slot-scope="scope">{{scope.row.id}}</template>
        </el-table-column>
        <el-table-column label="Image" width="120" align="center">
          <template slot-scope="scope">
            <img style="height: 80px" :src="scope.row.pic">
          </template>
        </el-table-column>
        <el-table-column label="Product Name" align="center">
          <template slot-scope="scope">
            <p>{{scope.row.name}}</p>
            <p>Brand: {{scope.row.brandName}}</p>
          </template>
        </el-table-column>
        <el-table-column label="Price / Code" width="120" align="center">
          <template slot-scope="scope">
            <p>Price: ￥{{scope.row.price}}</p>
            <p>Code: {{scope.row.productSn}}</p>
          </template>
        </el-table-column>
        <el-table-column label="Tags" width="140" align="center">
          <template slot-scope="scope">
            <p>Published:
              <el-switch
                @change="handlePublishStatusChange(scope.$index, scope.row)"
                :active-value="1"
                :inactive-value="0"
                v-model="scope.row.publishStatus">
              </el-switch>
            </p>
            <p>New:
              <el-switch
                @change="handleNewStatusChange(scope.$index, scope.row)"
                :active-value="1"
                :inactive-value="0"
                v-model="scope.row.newStatus">
              </el-switch>
            </p>
            <p>Recommended:
              <el-switch
                @change="handleRecommendStatusChange(scope.$index, scope.row)"
                :active-value="1"
                :inactive-value="0"
                v-model="scope.row.recommandStatus">
              </el-switch>
            </p>
          </template>
        </el-table-column>
        <el-table-column label="Sort" width="100" align="center">
          <template slot-scope="scope">{{scope.row.sort}}</template>
        </el-table-column>
        <el-table-column label="SKU Stock" width="100" align="center">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              @click="handleShowSkuEditDialog(scope.$index, scope.row)"
              circle>
            </el-button>
          </template>
        </el-table-column>
        <el-table-column label="Sales" width="100" align="center">
          <template slot-scope="scope">{{scope.row.sale}}</template>
        </el-table-column>
        <el-table-column label="Verification Status" width="100" align="center">
          <template slot-scope="scope">
            <p>{{scope.row.verifyStatus | verifyStatusFilter}}</p>
            <p>
              <el-button
                type="text"
                @click="handleShowVerifyDetail(scope.$index, scope.row)">
                View Details
              </el-button>
            </p>
          </template>
        </el-table-column>
        <el-table-column label="Actions" width="160" align="center">
          <template slot-scope="scope">
            <p>
              <el-button size="mini" @click="handleShowProduct(scope.$index, scope.row)">View</el-button>
              <el-button size="mini" @click="handleUpdateProduct(scope.$index, scope.row)">Edit</el-button>
            </p>
            <p>
              <el-button size="mini" @click="handleShowLog(scope.$index, scope.row)">Logs</el-button>
              <el-button size="mini" type="danger" @click="handleDelete(scope.$index, scope.row)">Delete</el-button>
            </p>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="batch-operate-container">
      <el-select size="small" v-model="operateType" placeholder="Batch Actions">
        <el-option
          v-for="item in operates"
          :key="item.value"
          :label="item.label"
          :value="item.value">
        </el-option>
      </el-select>
      <el-button
        style="margin-left: 20px"
        class="search-button"
        @click="handleBatchOperate()"
        type="primary"
        size="small">
        Confirm
      </el-button>
    </div>

    <div class="pagination-container">
      <el-pagination
        background
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        layout="total, sizes, prev, pager, next, jumper"
        :page-size="listQuery.pageSize"
        :page-sizes="[5,10,15]"
        :current-page.sync="listQuery.pageNum"
        :total="total">
      </el-pagination>
    </div>

    <el-dialog
      title="Edit SKU Information"
      :visible.sync="editSkuInfo.dialogVisible"
      width="40%">
      <span>Product Code: </span>
      <span>{{editSkuInfo.productSn}}</span>
      <el-input
        placeholder="Search by SKU Code"
        v-model="editSkuInfo.keyword"
        size="small"
        style="width: 50%;margin-left: 20px">
        <el-button slot="append" icon="el-icon-search" @click="handleSearchEditSku"></el-button>
      </el-input>
      <el-table style="width: 100%;margin-top: 20px" :data="editSkuInfo.stockList" border>
        <el-table-column label="SKU Code" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.skuCode"></el-input>
          </template>
        </el-table-column>
        <el-table-column
          v-for="(item,index) in editSkuInfo.productAttr"
          :label="item.name"
          :key="item.id"
          align="center">
          <template slot-scope="scope">
            {{getProductSkuSp(scope.row,index)}}
          </template>
        </el-table-column>
        <el-table-column label="Price" width="80" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.price"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="Stock" width="80" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.stock"></el-input>
          </template>
        </el-table-column>
        <el-table-column label="Low Stock Alert" width="100" align="center">
          <template slot-scope="scope">
            <el-input v-model="scope.row.lowStock"></el-input>
          </template>
        </el-table-column>
      </el-table>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editSkuInfo.dialogVisible = false">Cancel</el-button>
        <el-button type="primary" @click="handleEditSkuConfirm">Confirm</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
  import {
    fetchList,
    updateDeleteStatus,
    updateNewStatus,
    updateRecommendStatus,
    updatePublishStatus
  } from '@/api/product'
  import {fetchList as fetchSkuStockList,update as updateSkuStockList} from '@/api/skuStock'
  import {fetchList as fetchProductAttrList} from '@/api/productAttr'
  import {fetchList as fetchBrandList} from '@/api/brand'
  import {fetchListWithChildren} from '@/api/productCate'
  const defaultListQuery = {
    keyword: null,
    pageNum: 1,
    pageSize: 5,
    publishStatus: null,
    verifyStatus: null,
    productSn: null,
    productCategoryId: null,
    brandId: null
  };
  export default {
    name: "productList",
    data() {
      return {
        editSkuInfo:{
          dialogVisible:false,
          productId:null,
          productSn:'',
          productAttributeCategoryId:null,
          stockList:[],
          productAttr:[],
          keyword:null
        },
      operates: [
        { label: "Publish Product", value: "publishOn" },
        { label: "Unpublish Product", value: "publishOff" },
        { label: "Mark as Recommended", value: "recommendOn" },
        { label: "Unmark Recommended", value: "recommendOff" },
        { label: "Mark as New", value: "newOn" },
        { label: "Unmark New", value: "newOff" },
        { label: "Move to Category", value: "transferCategory" },
        { label: "Move to Recycle Bin", value: "recycle" }
      ],
      operateType: null,
      listQuery: Object.assign({}, defaultListQuery),
      list: null,
      total: null,
      listLoading: true,
      selectProductCateValue: null,
      multipleSelection: [],
      productCateOptions: [],
      brandOptions: [],
      publishStatusOptions: [
        { value: 1, label: 'Published' },
        { value: 0, label: 'Unpublished' }
      ],
      verifyStatusOptions: [
        { value: 1, label: 'Verified' },
        { value: 0, label: 'Unverified' }
      ]
    }
  },
  created() {
      this.getList();
      this.getBrandList();
      this.getProductCateList();
    },
    watch: {
      selectProductCateValue: function (newValue) {
        if (newValue != null && newValue.length == 2) {
          this.listQuery.productCategoryId = newValue[1];
        } else {
          this.listQuery.productCategoryId = null;
        }

      }
    },
  filters: {
    verifyStatusFilter(value) {
      return value === 1 ? 'Verified' : 'Unverified';
    }
  },
  methods: {
      getProductSkuSp(row, index) {
        let spData = JSON.parse(row.spData);
        if(spData!=null&&index<spData.length){
          return spData[index].value;
        }else{
          return null;
        }
      },
      getList() {
        this.listLoading = true;
        fetchList(this.listQuery).then(response => {
          this.listLoading = false;
          this.list = response.data.list;
          this.total = response.data.total;
        });
      },
      getBrandList() {
        fetchBrandList({pageNum: 1, pageSize: 100}).then(response => {
          this.brandOptions = [];
          let brandList = response.data.list;
          for (let i = 0; i < brandList.length; i++) {
            this.brandOptions.push({label: brandList[i].name, value: brandList[i].id});
          }
        });
      },
      getProductCateList() {
        fetchListWithChildren().then(response => {
          let list = response.data;
          this.productCateOptions = [];
          for (let i = 0; i < list.length; i++) {
            let children = [];
            if (list[i].children != null && list[i].children.length > 0) {
              for (let j = 0; j < list[i].children.length; j++) {
                children.push({label: list[i].children[j].name, value: list[i].children[j].id});
              }
            }
            this.productCateOptions.push({label: list[i].name, value: list[i].id, children: children});
          }
        });
      },
      handleShowSkuEditDialog(index,row){
        this.editSkuInfo.dialogVisible=true;
        this.editSkuInfo.productId=row.id;
        this.editSkuInfo.productSn=row.productSn;
        this.editSkuInfo.productAttributeCategoryId = row.productAttributeCategoryId;
        this.editSkuInfo.keyword=null;
        fetchSkuStockList(row.id,{keyword:this.editSkuInfo.keyword}).then(response=>{
          this.editSkuInfo.stockList=response.data;
        });
        if(row.productAttributeCategoryId!=null){
          fetchProductAttrList(row.productAttributeCategoryId,{type:0}).then(response=>{
            this.editSkuInfo.productAttr=response.data.list;
          });
        }
      },
      handleSearchEditSku(){
        fetchSkuStockList(this.editSkuInfo.productId,{keyword:this.editSkuInfo.keyword}).then(response=>{
          this.editSkuInfo.stockList=response.data;
        });
      },
    handleEditSkuConfirm() {
      if (!this.editSkuInfo.stockList || this.editSkuInfo.stockList.length <= 0) {
        this.$message({
          message: 'No SKU info available',
          type: 'warning',
          duration: 1000
        });
        return;
      }
      this.$confirm('Confirm to save changes?', 'Notice', {
        confirmButtonText: 'Confirm',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }).then(() => {
        updateSkuStockList(this.editSkuInfo.productId, this.editSkuInfo.stockList).then(response => {
          this.$message({
            message: 'Updated successfully',
            type: 'success',
            duration: 1000
          });
          this.editSkuInfo.dialogVisible = false;
        });
      });
    },
    handleSearchList() {
        this.listQuery.pageNum = 1;
        this.getList();
      },
      handleAddProduct() {
        this.$router.push({path:'/pms/addProduct'});
      },
    handleBatchOperate() {
      if (this.operateType == null) {
        this.$message({
          message: 'Please select an operation type',
          type: 'warning',
          duration: 1000
        });
        return;
      }
      if (!this.multipleSelection || this.multipleSelection.length <1) {
        this.$message({
          message: 'Please select products to operate',
          type: 'warning',
          duration: 1000
        });
        return;
      }
      this.$confirm('Are you sure to perform this batch operation?', 'Notice', {
        confirmButtonText: 'Confirm',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }).then(() => {
        let ids = this.multipleSelection.map(item => item.id);
        switch (this.operateType) {
          case this.operates[0].value:
            this.updatePublishStatus(1, ids);
            break;
          case this.operates[1].value:
            this.updatePublishStatus(0, ids);
            break;
          case this.operates[2].value:
            this.updateRecommendStatus(1, ids);
            break;
          case this.operates[3].value:
            this.updateRecommendStatus(0, ids);
            break;
          case this.operates[4].value:
            this.updateNewStatus(1, ids);
            break;
          case this.operates[5].value:
            this.updateNewStatus(0, ids);
            break;
          case this.operates[6].value:
            break;
          case this.operates[7].value:
            this.updateDeleteStatus(1, ids);
            break;
          default:
            break;
        }
        this.getList();
      });
    },
    handleSizeChange(val) {
        this.listQuery.pageNum = 1;
        this.listQuery.pageSize = val;
        this.getList();
      },
      handleCurrentChange(val) {
        this.listQuery.pageNum = val;
        this.getList();
      },
      handleSelectionChange(val) {
        this.multipleSelection = val;
      },
      handlePublishStatusChange(index, row) {
        let ids = [];
        ids.push(row.id);
        this.updatePublishStatus(row.publishStatus, ids);
      },
      handleNewStatusChange(index, row) {
        let ids = [];
        ids.push(row.id);
        this.updateNewStatus(row.newStatus, ids);
      },
      handleRecommendStatusChange(index, row) {
        let ids = [];
        ids.push(row.id);
        this.updateRecommendStatus(row.recommandStatus, ids);
      },
      handleResetSearch() {
        this.selectProductCateValue = [];
        this.listQuery = Object.assign({}, defaultListQuery);
      },
    handleDelete(index, row) {
      this.$confirm('Are you sure you want to delete this item?', 'Notice', {
        confirmButtonText: 'Confirm',
        cancelButtonText: 'Cancel',
        type: 'warning'
      }).then(() => {
        let ids = [];
          ids.push(row.id);
          this.updateDeleteStatus(1,ids);
        });
      },
      handleUpdateProduct(index,row){
        this.$router.push({path:'/pms/updateProduct',query:{id:row.id}});
      },
      handleShowProduct(index,row){
        console.log("handleShowProduct",row);
      },
      handleShowVerifyDetail(index,row){
        console.log("handleShowVerifyDetail",row);
      },
      handleShowLog(index,row){
        console.log("handleShowLog",row);
      },
    updatePublishStatus(publishStatus, ids) {
      let params = new URLSearchParams();
      params.append('ids', ids);
      params.append('publishStatus', publishStatus);
      updatePublishStatus(params).then(() => {
        this.$message({
          message: 'Updated successfully',
          type: 'success',
          duration: 1000
        });
      });
    },
    updateNewStatus(newStatus, ids) {
      let params = new URLSearchParams();
      params.append('ids', ids);
      params.append('newStatus', newStatus);
      updateNewStatus(params).then(() => {
        this.$message({
          message: 'Updated successfully',
          type: 'success',
          duration: 1000
        });
      });
    },
    updateRecommendStatus(recommendStatus, ids) {
      let params = new URLSearchParams();
      params.append('ids', ids);
      params.append('recommendStatus', recommendStatus);
      updateRecommendStatus(params).then(() => {
        this.$message({
          message: 'Updated successfully',
          type: 'success',
          duration: 1000
        });
      });
    },
    updateDeleteStatus(deleteStatus, ids) {
        let params = new URLSearchParams();
        params.append('ids', ids);
        params.append('deleteStatus', deleteStatus);
        updateDeleteStatus(params).then(response => {
          this.$message({
            message: 'Delete successfully',
            type: 'success',
            duration: 1000
          });
        });
        this.getList();
    }
  }
}
</script>
<style></style>


