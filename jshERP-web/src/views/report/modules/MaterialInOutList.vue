<template>
  <div ref="container">
    <a-modal
      :title="title"
      :width="1200"
      :visible="visible"
      :getContainer="() => $refs.container"
      :maskStyle="{'top':'93px','left':'154px'}"
      :wrapClassName="wrapClassNameInfo()"
      :mask="isDesktop()"
      :maskClosable="false"
      @cancel="handleCancel"
      cancelText="关闭"
      style="top:20px;height: 95%;">
      <template slot="footer">
        <a-button key="back" @click="handleCancel">取消</a-button>
      </template>
      <!-- table区域-begin -->
      <a-table
        bordered
        ref="table"
        size="middle"
        rowKey="id"
        :columns="columns"
        :dataSource="dataSource"
        :pagination="ipagination"
        :loading="loading"
        @change="handleTableChange">
        <span slot="numberCustomRender" slot-scope="text, record">
          <a @click="myHandleDetail(record)">{{record.number}}</a>
        </span>
      </a-table>
      <!-- table区域-end -->
      <!-- 表单区域 -->
      <bill-detail ref="billDetail"></bill-detail>
    </a-modal>
  </div>
</template>
<script>
  import BillDetail from '../../bill/dialog/BillDetail'
  import { JeecgListMixin } from '@/mixins/JeecgListMixin'
  import JEllipsis from '@/components/jeecg/JEllipsis'
  import {findBillDetailByNumber, findFinancialDetailByNumber} from '@/api/api'
  import {mixinDevice} from '@/utils/mixin'
  export default {
    name: "MaterialInOutList",
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      BillDetail,
      JEllipsis
    },
    data () {
      return {
        title:"操作",
        visible: false,
        disableMixinCreated: false,
        toFromType: '',
        // 查询条件
        queryParam: {
          depotIds: '',
          materialId:'',
        },
        tabKey: "1",
        // 表头
        columns: [
          {
            title: '#',
            dataIndex: '',
            key:'rowIndex',
            width:40,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title: '单据编号', dataIndex: 'number', width: 100,
            scopedSlots: { customRender: 'numberCustomRender' },
          },
          { title: '类型', dataIndex: 'type', width: 100},
          { title: '条码', dataIndex: 'barCode', width: 100},
          { title: '名称', dataIndex: 'materialName', width: 200},
          { title: '仓库名称', dataIndex: 'depotName', width: 80},
          { title: '数量', dataIndex: 'basicNumber', width: 70},
          { title: '日期', dataIndex: 'operTime', width: 110}
        ],
        labelCol: {
          xs: { span: 1 },
          sm: { span: 2 },
        },
        wrapperCol: {
          xs: { span: 10 },
          sm: { span: 16 },
        },
        url: {
          list: "/depotItem/findDetailByDepotIdsAndMaterialId"
        }
      }
    },
    created() {
    },
    methods: {
      getQueryParams() {
        let param = Object.assign({}, this.queryParam, this.isorter);
        param.field = this.getQueryField();
        param.currentPage = this.ipagination.current;
        param.pageSize = this.ipagination.pageSize;
        return param;
      },
      show(record, depotIds) {
        this.model = Object.assign({}, record);
        this.visible = true;
        this.queryParam.depotIds = depotIds
        this.queryParam.materialId = record.id
        this.loadData(1)
      },
      close () {
        this.$emit('close');
        this.visible = false;
      },
      handleCancel () {
        this.close()
      },
      myHandleDetail(record) {
        let that = this
        this.toFromType = record.fromType
        findBillDetailByNumber({ number: record.number }).then((res) => {
          if (res && res.code === 200) {
            this.$refs.billDetail.isCanBackCheck = false
            that.$refs.billDetail.show(res.data, record.type);
            that.$refs.billDetail.title="详情";
          }
        })
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less'
</style>