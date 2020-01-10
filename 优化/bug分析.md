## 内存泄漏

1.业绩统计

```vue
<Col style="width:250px">
                    <FormItem label="部门">
                        <Select v-model="queryInfo.deptName">
                            <Option value="all">全部</Option>
                            <Option v-for="dict in departmentDicts" :key="dict.dictId" :value="dict.dictValue">{{
                                dict.dictLabel
                            }}</Option>
                        </Select>
                    </FormItem>
                </Col>
```

该部分会引起内存泄漏。

分析：

1. 切换成element组件，同样有问题，排除组件问题。

2. 将departmentDicts不从接口获取，直接写死在data，没有问题。

3. 对getDict方法进行分析

   ```js
   async getDicts() {
               this.departmentDicts = [];
               const { data } = await getBusinessDeptDicts();
               if (data.success && data.data) {
                   this.departmentDicts = JSON.parse(JSON.stringify(data.data));
               }
           }
   ```

   经测试，this.departmentDicts=[]有问题。去除后问题解决。

   原因待分析。

2.工作流

同样，去掉headerSearch组件后正常。

分析

```vue
<Col :lg="10" :xl="10" :xxl="7">
                    <FormItem label="时间区间">
                        <DatePicker
                            type="date"
                            v-model="storeState.beginDate"
                            style="width:120px"
                            placeholder="开始时间"
                            :options="beginDateOptions"
                        />
                        &nbsp;~&nbsp;
                        <DatePicker
                            type="date"
                            v-model="storeState.endDate"
                            style="width:120px"
                            placeholder="结束时间"
                            :options="endDateOptions"
                        />
                    </FormItem>
                </Col>

```

```js
data() {
        return {
            storeState: store.state,
            dictMap,
            beginDateOptions: {
                disabledDate: date => {
                    if (!this.storeState.endDate) {
                        return false;
                    } else {
                        return date && dayjs(date).diff(dayjs(this.storeState.endDate)) > 0;
                    }
                }
            },
            endDateOptions: {
                disabledDate: date => {
                    if (!this.storeState.beginDate) {
                        return false;
                    } else {
                        return date && dayjs(date).diff(dayjs(this.storeState.beginDate)) < 0;
                    }
                }
            }
        };
    },
```



去掉该部分正常。

怀疑endDateoptions的问题，去掉options，无效。只有去掉storestate.beginDate才有效。

但是其他地方的datepicker这么用并没有问题。

替换datepicker组件，问题依旧，排除组件问题。

发现index.vue中每次mounted时initData会对date重新赋值

```js
initData() {
            this.routeName = this.$route.name;
            // const now = dayjs().endOf('day');
            // const prev = now.subtract(1, 'month');
            // this.storeState.beginDate = prev.toDate();
            // this.storeState.endDate = now.toDate();
            this.getData();
            this.filterColumns();
        },
```



```
$route() {
            store.clear();
            this.initData();
        }
```

clear时会重新赋值date数据，initData时又重新赋值，可能这个原因引起。和上面一样。

3。客户管理

4.权限管理