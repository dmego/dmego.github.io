---
layout: post
title: "fastJosn使用总结"
date: 2018-05-13 19:56
comments: true
categories: [java学习笔记]
tags: [json]
copyright: true
---

<center>
<img src="http://ovasw3yf9.bkt.clouddn.com/blog/180513/cE7H2aLhmC.jpg?imageslim" width="250px" />
</center>

## 前言
最近在开发过程中使用了大量的`json`作为前后端数据交换的方式,由于之前没有对`json`做过系统的学习,所有在使用过程中查阅了大量的文档与资料,这里主要记录了我在开发后对`json`以及`fastjson`使用的总结

## JSON 介绍
`JSON`(javaScript Object Notation)是一种轻量级的数据交换格式。主要采用键值对(`{"name": "json"}`)的方式来保存和表示数据。`JSON`是`JS`对象的字符串表示法，它使用文本表示一个`JS`对象的信息，本质上是一个字符串。更多简介见[介绍JSON](http://www.json.org/json-zh.html)。

## fastjson 简介
在日志解析,前后端数据传输交互中,经常会遇到字符串(String)与`json`,`XML`等格式相互转换与解析，其中`json`以跨语言，跨前后端的优点在开发中被频繁使用，基本上可以说是标准的数据交换格式。[fastjson](https://github.com/alibaba/fastjson)是一个java语言编写的高性能且功能完善的JSON库，它采用一种“假定有序快速匹配”的算法，把`JSON Parse` 的性能提升到了极致。它的接口简单易用，已经被广泛使用在缓存序列化，协议交互，Web输出等各种应用场景中。
<!--more -->

## fastjson 常用 API
fastjson API 入口类是`com.alibaba.fastjson.JSON`,常用的序列化操作都可以在`JSON`类上的静态方法直接完成。
```java
public static final Object parse(String text); // 把JSON文本parse为JSONObject或者JSONArray 
public static final JSONObject parseObject(String text)； // 把JSON文本parse成JSONObject    
public static final <T> T parseObject(String text, Class<T> clazz); // 把JSON文本parse为JavaBean 
public static final JSONArray parseArray(String text); // 把JSON文本parse成JSONArray 
public static final <T> List<T> parseArray(String text, Class<T> clazz); //把JSON文本parse成JavaBean集合 
public static final String toJSONString(Object object); // 将JavaBean序列化为JSON文本 
public static final String toJSONString(Object object, boolean prettyFormat); // 将JavaBean序列化为带格式的JSON文本 
public static final Object toJSON(Object javaObject); //将JavaBean转换为JSONObject或者JSONArray。
```
### 使用方法举例
```java
//将JSON文本转换为java对象
import com.alibaba.fastjson.JSON;
Model model = JSON.parseObject(jsonStr, Model.class);
```
### 有关类库的一些说明
- JSONArray : 相当于List<Object>
- JSONObject: 相当于Map<String,Object>


## fastjson 使用实例

### java对象与JSON字符串的互转

 User测试类
```java
/**
 * User测试类
 * @author dmego
 */
public class User {
	private String username;
	private String password;
	public User(){}
	public User(String username,String password){
		this.username = username;
		this.password = password;
	}
	public String getUsername() {
		return username;
	}
	public void setUsername(String username) {
		this.username = username;
	}
	public String getPassword() {
		return password;
	}
	public void setPassword(String password) {
		this.password = password;
	}
    @Override
	public String toString() {
		return "User [username=" + username + ", password=" + password + "]";
	}
}
```
UserGroup测试类
```java
import java.util.ArrayList;
import java.util.List;

/**
 * 用户组测试类
 * @author dmego
 *
 */
public class UserGroup {
	private String name;  
    private List<User> users = new ArrayList<User>();
    public UserGroup(){}
    public UserGroup(String name,List<User> users){
    	this.name = name;
    	this.users = users;
    }
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	public List<User> getUsers() {
		return users;
	}
	public void setUsers(List<User> users) {
		this.users = users;
	}
	@Override
	public String toString() {
		return "UserGroup [name=" + name + ", users=" + users + "]";
	}  
}
```
fastJson测试类
```java
package demo;
import java.util.ArrayList;
import java.util.List;
import org.junit.Test;
import com.alibaba.fastjson.JSON;

/**
 * fastJson测试类
 * @author dmego
 *
 */
public class TestFastJosn {
	/**
	 * java对象转 json字符串 
	 */
	@Test
	public void objectTOJson(){
		//简单java类转json字符串
		User user = new User("dmego", "123456");
		String UserJson = JSON.toJSONString(user);
		System.out.println("简单java类转json字符串:"+UserJson);
		
		//List<Object>转json字符串
		User user1 = new User("zhangsan", "123123");
		User user2 = new User("lisi", "321321");
		List<User> users = new ArrayList<User>();
		users.add(user1);
		users.add(user2);
		String ListUserJson = JSON.toJSONString(users);
		System.out.println("List<Object>转json字符串:"+ListUserJson);	
		
		//复杂java类转json字符串
		UserGroup userGroup = new UserGroup("userGroup", users);
		String userGroupJson = JSON.toJSONString(userGroup);
		System.out.println("复杂java类转json字符串:"+userGroupJson);		
		
	}
	
	/**
	 * json字符串转java对象
	 * 注：字符串中使用双引号需要转义 (" --> \"),这里使用的是单引号
	 */
	@Test
	public void JsonTOObject(){
		/* json字符串转简单java对象
	     * 字符串：{"password":"123456","username":"dmego"}*/
		
		String jsonStr1 = "{'password':'123456','username':'dmego'}";
		User user = JSON.parseObject(jsonStr1, User.class);
		System.out.println("json字符串转简单java对象:"+user.toString());
		
		/*
		 * json字符串转List<Object>对象
		 * 字符串：[{"password":"123123","username":"zhangsan"},{"password":"321321","username":"lisi"}]
		 */
		String jsonStr2 = "[{'password':'123123','username':'zhangsan'},{'password':'321321','username':'lisi'}]";
		List<User> users = JSON.parseArray(jsonStr2, User.class);
		System.out.println("json字符串转List<Object>对象:"+users.toString());
			
		/*json字符串转复杂java对象
		 * 字符串：{"name":"userGroup","users":[{"password":"123123","username":"zhangsan"},{"password":"321321","username":"lisi"}]}
		 * */
		String jsonStr3 = "{'name':'userGroup','users':[{'password':'123123','username':'zhangsan'},{'password':'321321','username':'lisi'}]}";
		UserGroup userGroup = JSON.parseObject(jsonStr3, UserGroup.class);
		System.out.println("json字符串转复杂java对象:"+userGroup);	
	}
}
```
输出结果
```java
简单java类转json字符串:{"password":"123456","username":"dmego"}
List<Object>转json字符串:[{"password":"123123","username":"zhangsan"},{"password":"321321","username":"lisi"}]
复杂java类转json字符串:{"name":"userGroup","users":[{"password":"123123","username":"zhangsan"},{"password":"321321","username":"lisi"}]}

json字符串转简单java对象:User [username=dmego, password=123456]
json字符串转List<Object>对象:[User [username=zhangsan, password=123123], User [username=lisi, password=321321]]
json字符串转复杂java对象:UserGroup [name=userGroup, users=[User [username=zhangsan, password=123123], User [username=lisi, password=321321]]]
```

### fastjson 解析复杂嵌套json字符串
这个实例是我在开发中用到的，先给出要解析的json字符串
```json
[
    {
        "id": "user_list",
        "key": "id",
        "tableName": "用户列表",
        "className": "cn.dmego.domain.User",
        "column": [
            {
                "key": "rowIndex",
                "header": "序号",
                "width": "50",
                "allowSort": "false"
            },
            {
                "key": "id",
                "header": "id",
                "hidden": "true"
            },
            {
                "key": "name",
                "header": "姓名",
                "width": "100",
                "allowSort": "true"
            }
        ]
    },
    {
        "id": "role_list",
        "key": "id",
        "tableName": "角色列表",
        "className": "cn.dmego.domain.Role",
        "column": [
            {
                "key": "rowIndex",
                "header": "序号",
                "width": "50",
                "allowSort": "false"
            },
            {
                "key": "id",
                "header": "id",
                "hidden": "true"
            },
            {
                "key": "name",
                "header": "名称",
                "width": "100",
                "allowSort": "true"
            }
        ]
    }
]
```
要想解析这种复杂的字符串，首先得先定义好与之相符的java POJO 对象，经过观察，我们发现，这个是一个json对象数组，每一个对象里包含了许多属性，其中还有一个属性的类型也是对象数组。所有，我们从里到外，先定义最里面的对象：
```java
public class Column {
	private String key;
	private String header;
	private String width;
	private String allowSort;
	private String hidden;
	
	public String getKey() {
		return key;
	}
	public void setKey(String key) {
		this.key = key;
	}
    //这里省略部分getter与setter方法	
}
```
再定义外层的对象：
```java
import java.util.List;
import org.apache.commons.collections4.map.LinkedMap;

public class Query {
	private String id;
	private String key;
	private String tableName;
	private String className;
    private List<LinkedMap<String, Object>> column; 
    private List<Column> columnList;
    
	public String getId() {
		return id;
	}
	public void setId(String id) {
		this.id = id;
	}
    //这里省略部分getter与setter方法	
	public List<LinkedMap<String, Object>> getColumn() {
		return column;
	}
	public void setColumn(List<LinkedMap<String, Object>> column) {
		this.column = column;
	}
	public List<Column> getColumnList() {
		return columnList;
	}
	public void setColumnList(List<Column> columnList) {
		this.columnList = columnList;
	}
}
```

我的这个json文件放置在类路径下，最后想将这个json字符串转化为List<Query>对象，并且将column 对象数组转化为query对象里的List<Column>属性
而实际转化过程中，fastjson将column对象数组转化为List<Map>;所有我们还需要将Map类型转化为object类型才能满足需求。
```java
    /**
	 * 读取类路径下的配置文件
	 * 解析成对象数组并返回
	 * @throws IOException
	 */
	@Test
	public List<Query> test() throws IOException {
		// 读取类路径下的query.json文件
		ClassLoader cl = this.getClass().getClassLoader();
		InputStream inputStream = cl.getResourceAsStream("query.json");
		String jsontext = IOUtils.toString(inputStream, "utf8");
		// 先将字符jie串转为List数组
		List<Query> queryList = JSON.parseArray(jsontext, Query.class);
		for (Query query : queryList) {
			List<Column> columnList = new ArrayList<Column>();
			List<LinkedMap<String,Object>> columns = query.getColumn();
			for (LinkedMap<String, Object> linkedMap : columns) {
				//将map转化为java实体类
				Column column = (Column)map2Object(linkedMap, Column.class);
				System.out.println(column.toString());
				columnList.add(column);
			}
			query.setColumnList(columnList); //为columnList属性赋值
		}
        return queryList;
	}

    /**
     * Map转成实体对象
     * @param map map实体对象包含属性
     * @param clazz 实体对象类型
     * @return
     */
    public static Object map2Object(Map<String, Object> map, Class<?> clazz) {
        if (map == null) {
            return null;
        }
        Object obj = null;
        try {
            obj = clazz.newInstance();
            Field[] fields = obj.getClass().getDeclaredFields();
            for (Field field : fields) {
                int mod = field.getModifiers();
                if (Modifier.isStatic(mod) || Modifier.isFinal(mod)) {
                    continue;
                }
                field.setAccessible(true);
                String flag = (String) map.get(field.getName());
                if(flag != null){
                	if(flag.equals("false") || flag.equals("true")){
                    	field.set(obj, Boolean.parseBoolean(flag));
                    }else{
                    	field.set(obj, map.get(field.getName()));
                    }
                }                
            }
        } catch (Exception e) {
            e.printStackTrace();
        } 
        return obj;
    }

```

## 参考
- [fastJson](https://github.com/alibaba/fastjson)
- [JSON最佳实践](http://kimmking.github.io/2017/06/06/json-best-practice/#9-fastjson_u7684_u6700_u4F73_u5B9E_u8DF5)
- [介绍 JSON](http://www.json.org/json-zh.html)
- [W3Cschool FastJson 教程](https://www.w3cschool.cn/fastjson/)
