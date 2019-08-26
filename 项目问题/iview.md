*19-8-26*

iview分页中每次pagesize-change都会引起page-change

解决方法：

- pageSizeChange:设置page=1，pagesize=pagesize，因为当page=3，pagesize=100时，可能没有对应的数据，~~此时total为0，只会显示一个分页，无法切换到其他有数据的页面~~,此时total还是原来值如1000，但如果pagesize=9999，只会有一个分页.如果此时返回的page=2的数据，那么table是没有数据的，如果显示的page=2，那么将无法切换，_因此此时应该是page=1，pagesize=9999的数据_。
- 每次page-size-change时，会导致pagechange且page=1，此时会调用page=3，pagesize=100和page=1，pagesize=this.pagesize两个接口，组件如何显示取决于两个接口的返回速度，所以会出现两次显示结果不一致的情况。

因此每次获取数据时，先设置store的page，再在on-page-change里判断两个page是否一致，一致再调取数据。

尤其pageSize变化时，应使page=1.

```
handlePageChange(page) {
    if (page === this.page) return;
    this.getData({ page, pageSize: this.pageSize });
},
handlePageSizeChange(pageSize) {
    this.getData({ page: 1, pageSize });
},
```

