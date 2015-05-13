## ERP需求分析流程

1. **采购流程**

```flow
下采购单→采购审批→运输任务指派→判定供应商是否送货→运输任务完成→按采购订单任务入库→流程结束
```


```flow
st=>start: Start
e=>end
op=>operation: My Operation
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```