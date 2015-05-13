```java
package cn.itcast.invoice.auth.dep.web;


import java.util.List;

import javax.annotation.Resource;

import cn.itcast.invoice.auth.dep.business.ebi.DepEbi;
import cn.itcast.invoice.auth.dep.vo.DepModel;
import cn.itcast.invoice.auth.dep.vo.DepQueryModel;

import com.opensymphony.xwork2.ActionContext;
import com.opensymphony.xwork2.ActionSupport;

public class DepAction extends ActionSupport {
	//注入business
	@Resource(name="depEbo")
	private DepEbi depEbo;
	//属性驱动,接收部门信息参数
	public DepModel dep = new DepModel();
	//按条件查询
	public DepQueryModel dqm = new DepQueryModel();
	//接收分页数据
	public Integer pageNum;
	public Integer pageCount = 2;
	
	public String queryList(){
		
		List<DepModel> depList = depEbo.getAll(dqm);
		ActionContext.getContext().put("depList", depList);
		return "list";
	}
	/**
	 * 获取部门列表
	 * @return
	 */
	public String list() {
		//调用business 获取所有部门列表
		List<DepModel> depList = depEbo.getAll();
		//将查找到的所有部门信息存入值栈中
		ActionContext.getContext().put("depList", depList);
		return "list";
	}
	/**
	 * 保存部门
	 * @return "tolist"
	 */
	public String save() {
		//调用business保存部门信息
		depEbo.save(dep);
		return "tolist";
	}
}
```