## Hibernate 查询
dfgfdfg
*1.条件查询*
`hahahahhah11111111111111`

>hahahhaasdfasdf

```java
public List<DepModel> getAll(DepQueryModel dqm) {
	//使用QBC完成条件查询
	DetachedCriteria dc = DetachedCriteria.forClass(DepModel.class);
	//根据查询条件中封装的数据完成条件的拼接
	if(dqm.getName()!=null && dqm.getName().trim().length()>0){
		dc.add(Restrictions.like("name", "%"+dqm.getName().trim()+"%"));
	}
	if(dqm.getTele()!=null && dqm.getTele().trim().length()>0){
		dc.add(Restrictions.like("tele", "%"+dqm.getTele().trim()+"%"));
	}
	return this.getHibernateTemplate().findByCriteria(dc);
}
```

