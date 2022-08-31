```vue
 <a-table :columns="columns" :rowSelection="rowSelection" :dataSource="rentList" :pagination="RentPagination" @change="handleTableChange" :rowKey="record => record.appId">
      <span slot="action" slot-scope="text, record">{{text}}</span>
 </a-table>
```

```js
 rowSelection: {
    onChange: (selectedRowKeys, selectedRows) => this.onSelectChange(selectedRowKeys, selectedRows),
    onSelect: (record, selected, selectedRows) => {
                },
    onSelectAll: (selected, selectedRows, changeRows) => {
            },
  },
      
 pagination: {
                // 默认当前页，每页显示数量，总数
   current: 1, 
   pageSize: 100,
   total: 0,
   showSizeChanger: true, //可改变每页数量
   pageSizeOptions: ['10','20', '50', '100'],
   showTotal: total => `共 ${total} 条`,
   onChange: (current, pageSize) => this.pageChange(current, pageSize),//改变页面 比如第一页 第二页、、、
   onShowSizeChange: (current, pageSize) => this.pageSizeChange(current, pageSize)//选择要展示数据条数 比如10条每页
 },
```

```js
   onSelectChange(selectedRowKeys, selectedRows){
           
    },
        
  getList() {
      this.pagination.current = this.pagination.current?this.pagination.current:1;
      let url =  "/rent-platform/merchant/order/queryInvoiceList";
      var temp = {}
      temp.page = vm.pagination.current;
      temp.pageSize = vm.pagination.pageSize;
      this.axios.post(url,temp).then(res => {
          vm.pagination.list = res.data.result.list
          vm.pagination.total = res.data.result.total;
          vm.pagination.current = res.data.result.pageNum
      })
   },
       
       
       
  pageChange (current, pageSize) {
                this.pagination.current = current
                //下面再请求分页接口，重新渲染数据
   },
       
  pageSizeChange (current, pageSize) {
            this.pagination.pageSize = pageSize
            this.getList();
            // 请求分页接口，重新渲染数据
  },
 handleTableChange(pagination){
            this.pagination = pagination;
            this.getList()
 },
```

