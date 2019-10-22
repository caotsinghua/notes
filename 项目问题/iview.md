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

### input自动设置值

input在ie11无法初始化值

```
<FormItem label="日志主题" prop="serviceTitle">
                <Input v-model.trim="form.serviceTitle" placeholder="售前/售后+第N次拜访+其他重点" />
            </FormItem>
            <FormItem label="位置" prop="servicePosition">
                <Input v-model.trim="form.servicePosition" placeholder="11" />
            </FormItem>
```

日志主题无法获取值，位置可以

- type=textarea可以
- 去掉placeholder可以（但位置的input带placeholder是有值的）
- placeholder=售前/售后+第N次拜访+其他重点 就无法获取值
- 结论：**placeholder不能是中文** 

但是在其他页面中，placeholder为英文是可以的。

```
<EditForm ref="modal-edit-form" v-if="formVisible" />
```

- 去除formVisible可以显示
- 保留formvisible，下面这样也行

```
this.$nextTick(() => {
               setTimeout(()=>{
                  this.$refs['modal-edit-form'].initData(row);
               })
            });
```

- 发现在form组件的mounted中initData可以显示出数据

  **将initData的过程放到form的mounted中，解决问题。**

  ```
  // edit-form.vue
  mounted() {
          this.initData(this.editRow);
      },
  // edit-modal
  openModal(row) {
              this.visible = true;
              this.customerId = this.$route.params.customerId;
              this.editRow = row || null;
              this.customerLinkmanId = row && row.customerLinkmanId;
              this.readOnly = !!row.readOnly;
              this.$nextTick(() => {
                  this.formVisible = true;
              });
          },
  ```

  