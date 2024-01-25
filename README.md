基于ssm超市订单管理系统
环境：jdk1.8，mysql5.7，tomcat8.5，maven3.6
软件：IDEA
功能：超市后台管理系统，有订单管理，供应商管理，用户管理，密码修改，退出系统等模块
角色：管理员、经理、普通员工



# 环境要求

1.运行环境：最好是java jdk1.8,我们在这个平台上运行的。其他版本理论上也可以。 

2.IDE环境：IDEA,Eclipse,Myeclipse都可以。推荐IDEA; 

3.tomcat环境：Tomcat7.x,8.X,9.x版本均可 

4.硬件环境：windows7/8/10 4G内存以上；或者Mac OS; 

5.是否Maven项目：是；查看源码目录中是否包含pom.xml;若包含，则为maven项目，否则为非maven.项目 

6.数据库：MySql5.7/8.0等版本均可；

# 技术栈

后台框架：Spring Boot、MyBatis

数据库：MySQL

环境：JDK8、TOMCAT、IDEA

# 使用说明

1.使用Navicati或者其它工具，在mysql中创建对应sq文件名称的数据库，并导入项目的sql文件； 

2.使用IDEA/Eclipse/MyEclipse导入项目，修改配置，运行项目； 

3.将项目中config-propertiesi配置文件中的数据库配置改为自己的配置，然后运行；

# 运行指导

idea导入源码空间站顶目教程说明(Vindows版)-ssm篇：

http://mtw.so/5MHvZq 

源码地址：[http://codegym.top](http://codegym.top/)。 

# 运行截图
![微信截图_20240110232207](https://github.com/catnipculture/supermarketorder/assets/126983311/614e26ba-4a26-4b77-a080-a00c7e094ff0)
![微信截图_20240110232255](https://github.com/catnipculture/supermarketorder/assets/126983311/e6e6f779-ddba-4421-a27c-02678408e28c)
![微信截图_20240110232310](https://github.com/catnipculture/supermarketorder/assets/126983311/b062e3df-9c66-4359-bd92-64516305e6bf)
![微信截图_20240110232319](https://github.com/catnipculture/supermarketorder/assets/126983311/6da8c5ef-dcc0-4881-af1b-3264b5c76444)
![微信截图_20240110232328](https://github.com/catnipculture/supermarketorder/assets/126983311/bbf6ad37-5bcb-4c01-8108-3eee77506804)

## 代码
UserController 

```java
package com.yc.controller;

import java.util.Date;
import java.util.List;
import java.util.Map;

import javax.servlet.http.Cookie;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;

import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.JSONObject;
import com.github.pagehelper.PageInfo;
import com.yc.po.Role;
import com.yc.po.User;
import com.yc.service.UserService;
import com.yc.util.ResultData;
@RequestMapping("user")
@Controller
public class UserController {

	//业务对象
	@Autowired
	private UserService userService;
	//session域对象
	@Autowired
	private HttpSession session;
	
	/**
	 * 用户登录
	 * @param userCode
	 * @param password
	 * @return
	 */
	@RequestMapping("gologin")
	@ResponseBody
	public String gologin(String userCode,String password){
		ResultData rd = userService.goLogin(userCode, password);
		if(rd.getData() != null){
			//把用户信息存到session域,保存用户的在线状态
			session.setAttribute("user",rd.getData());
		}
		//返回json数据到客户端
		System.out.println(JSONObject.toJSONString(rd));
		return JSONObject.toJSONString(rd);
	}
	
	/**
	 * 退出系统功能
	 * @return
	 */
	@GetMapping("logout")
	public String logout(HttpServletResponse response){
		//删除session域里面用户信息
		session.removeAttribute("user");
		//删除cookie
		Cookie cookie = new Cookie("usercode","-1");
		cookie.setMaxAge(0);
		cookie.setPath("/");
		response.addCookie(cookie);
		return "redirect:/login.html";
	}
	
	/**
	 * 旧密码验证
	 * @param pwd
	 * @param id
	 * @return
	 */
	@PostMapping("isPwd")
	@ResponseBody
	public String isPwd(String pwd,Long id){
		ResultData rd = userService.isPwd(pwd, id);
		return JSONObject.toJSONString(rd);
	}
	
	/**
	 * 密码修改
	 * @param userpassword
	 * @param id
	 * @return
	 */
	@PostMapping("pwdupdate")
	@ResponseBody
	public String pwdupdate(String userpassword,Long id){
		int pwdset = userService.pwdupdate(userpassword, id);
		return JSONObject.toJSONString(pwdset);
	}
	
	/**
	 * 分页展示用户信息
	 * @param username
	 * @param rolename
	 * @param n
	 * @param pageSize
	 * @param map
	 * @return
	 */
	@GetMapping("userlist")
	public String getUserList(
			@RequestParam(value="username",defaultValue="")String username,
			@RequestParam(value="rolename",defaultValue="")String rolename,
			@RequestParam(value="n",defaultValue="1")Integer n,
			@RequestParam(value="pageSize",defaultValue="5")Integer pageSize,
			Map<String,Object> map){
		//获取用户列表信息
		PageInfo pageInfo = userService.getUserListPage(username,rolename,n, pageSize);
		//获取角色名称列表信息
		List<String> roleNames = userService.getRoleNames();
		map.put("pageInfo", pageInfo);
		map.put("roleNames",roleNames);
		map.put("username",username);
		map.put("rolename",rolename);
		return "userlist";
	}
	
	@ModelAttribute
	public void modelAttributeMethod(Long id,String flag,Map<String,Object> map){
		if("update".equals(flag)){
			User user = new User();
			user = userService.getUserById(id);
			
			List<Role> rnids = userService.getRoleNameAndIds();
			map.put("user",user);
			map.put("rnids",rnids);
		}
	}
	
	/**
	 * 跳转到用户更新页面
	 * @return
	 */
	@GetMapping("usermodify")
	public String usermodify(){
		return "usermodify";
	}
	
	/**
	 * 跳转到用户查看页面
	 * @return
	 */
	@GetMapping("userview")
	public String userview(){
		return "userview";
	}
	
	/**
	 * 跳转到用户增加页面
	 * @return
	 */
	@GetMapping("useradd")
	public String useradd(
			@RequestParam(value="rolename",defaultValue="")String rolename,
			Map<String,Object> map){
		//获取角色名称列表信息
		List<Role> roles = userService.getRoleNameAndIds();
		map.put("roles",roles);
		return "useradd";
	}
		
	/**
	 * 查看用户信息
	 * @return
	 */
	@RequestMapping("userselect")
	@ResponseBody
	public String userselect(Long id){			
			User user = userService.getUserById(id);			
			return JSON.toJSONString(user);		
	}
	
	/**
	 * 用户删除操作
	 * @return
	 */
	@RequestMapping("userdel")
	public String userdel(@RequestParam("id") Long id){
		userService.userDelect(id);
		return "redirect:userlist";
	}
		
	/**
	 * 更新用户信息
	 * @return
	 */
	@PostMapping("update")
	public String userupdate(User user,Map<String,Object> map){
		//修改时间
		user.setModifydate(new Date());
		//修改人
		User onlineUser = (User)session.getAttribute("user");
		if(onlineUser != null){
			user.setModifyby(onlineUser.getId());
		}
		userService.userUpdate(user);
		
		List<Role> rnids = userService.getRoleNameAndIds();
		map.put("rnids",rnids);
		return "usermodify";
	}
	
	/**
	 * 增加用户信息
	 * @return
	 */
	@PostMapping("add")
	public String useradd(User user,Map<String,Object> map){
	
		user.setCreationdate(new Date());		
		//修改人
		User onlineUser = (User)session.getAttribute("user");
		if(onlineUser != null){
			user.setCreatedby(onlineUser.getId());
		}
		userService.userAdd(user);
		return "redirect:userlist";
	}
	
	
}
```


ProviderController 

```java
package com.yc.controller;

import java.util.Date;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpSession;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.ModelAttribute;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;

import com.github.pagehelper.PageInfo;
import com.yc.po.Provider;
import com.yc.po.Role;
import com.yc.po.User;
import com.yc.service.ProviderService;

@RequestMapping("provider")
@Controller
public class ProviderController {
	
	@Autowired
	private ProviderService providerService;
	
	//session域对象
	@Autowired
	private HttpSession session;
	
	@GetMapping("providerlist")
	public String getProviderList(
			@RequestParam(value="proname",defaultValue="")String proname,
			@RequestParam(value="procontact",defaultValue="")String procontact,
			@RequestParam(value="n",defaultValue="1")Integer n,
			@RequestParam(value="pageSize",defaultValue="5")Integer pageSize,
			Map<String,Object> map){
		//获取用户列表信息
		PageInfo pageInfo = providerService.getProviderListPage(proname, procontact, n, pageSize);
		
		map.put("pageInfo", pageInfo);	
		map.put("proname",proname);
		map.put("procode",procontact);
		return "providerlist";
	}
	
	@ModelAttribute
	public void modelAttributeMethod(Long id,String flag,Map<String,Object> map){
		if("update".equals(flag)){
			Provider provider = new Provider();
			provider = providerService.getProById(id);
		
			map.put("provider",provider);
		}
	}
	
	/**
	 * 更新供应商信息
	 * @return
	 */
	@PostMapping("update")
	public String proupdate(Provider provider,Map<String,Object> map){
		//修改时间
		provider.setModifydate(new Date());
		//修改人
		User onlineUser = (User)session.getAttribute("user");
		if(onlineUser != null){
			provider.setModifyby(onlineUser.getId());
		}
		providerService.proUpdate(provider);
		
		return "redirect:providerlist";
	}
	
	/**
	 * 跳转到供应商更新页面
	 * @return
	 */
	@GetMapping("providermodify")
	public String providermodify(){
		return "providermodify";
	}
	
	/**
	 * 跳转到供应商查看页面
	 * @return
	 */
	@GetMapping("providerview")
	public String providerview(){
		return "providerview";
	}
	
	/**
	 * 跳转到供应商增加页面
	 * @return
	 */
	@GetMapping("provideradd")
	public String provideradd(){
		return "provideradd";
	}
	
	/**
	 * 供应商删除操作
	 * @return
	 */
	@RequestMapping("providerdel")
	public String userdel(@RequestParam("id") Long id){
		providerService.proDelect(id);
		return "redirect:providerlist";
	}
	
	/**
	 * 增加供应商信息
	 * @return
	 */
	@PostMapping("add")
	public String useradd(Provider provider,Map<String,Object> map){
	
		provider.setCreationdate(new Date());		
		//修改人
		User onlineUser = (User)session.getAttribute("user");
		if(onlineUser != null){
			provider.setCreatedby(onlineUser.getId());
		}
		providerService.proAdd(provider);
		return "redirect:providerlist";
	}
		
}
```
