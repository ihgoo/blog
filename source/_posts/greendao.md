title: GreenDao学习
date: 2014-10-29 21:38:22
tags:
---
因公司项目需求,需要使用到greendao,所以对其进行研究了一下


####greenDAO相关
- greenDAO官网：http://greendao-orm.com/
- 项目下载地址：https://github.com/greenrobot/greenDAO（或者官网）


greenDAO是一个可以帮助Android开发者快速将Java对象映射到SQLite数据库的表单中的ORM解决方案，通过使用一个简单的面向对象API，开发者可以对Java对象进行存储、更新、删除和查询。
greenDAO的主要设计目标：
<!--more-->
   * 最大性能（最快的Android ORM）
   * 易于使用API
   * 高度优化
   * 最小内存消耗


####使用greendao
官方Demo里共有六个工程目录，分别为：
- DaoCore：库目录，即jar文件greendao-1.3.0-beta-1.jar的代码；
- DaoExample：android范例工程；
- DaoExampleGenerator：DaoExample工程的DAO类构造器，java工程；
- DaoGenerator：DAO类构造器，java工程；
- DaoTest、PerformanceTestOrmLite：其他测试相关的工程。


###Dao类构造
首先需要新建一个java工程来生成dao类文件,该工程需要导入greendao-generator.jar和freemarker.jar文件到项目中。

````
package com.weijianzhi.daogenerator;

import de.greenrobot.daogenerator.DaoGenerator;
import de.greenrobot.daogenerator.Entity;
import de.greenrobot.daogenerator.Schema;

public class ExampleDaoGenerator {

	public static void main(String[] args) throws Exception {
		Schema schema = new Schema(1, "com.guangzhi.weijianzhi.dao");

		addTaskDetail(schema);

		new DaoGenerator().generateAll(schema, "../weijianzhi/src");
	}

	private static void addTaskDetail(Schema schema) {
		Entity entity = schema.addEntity("TaskDetail");
		entity.addIdProperty();
		entity.addStringProperty("uid");  //用户标识
		entity.addStringProperty("tid"); //任务id
		entity.addStringProperty("utid");
		entity.addStringProperty("tname");
		entity.addStringProperty("bonus");
		entity.addStringProperty("point");
		entity.addStringProperty("taskcontent");
		entity.addStringProperty("applylist");
		entity.addStringProperty("namelist");
		entity.addStringProperty("status");
	}

}
````

在main方法中
````
 Schema schema = new Schema(1, "com.guangzhi.weijianzhi.dao");
 //参数1为版本号 参数2表示所在包的路径
````
Entity entity = schema.addEntity("TaskDetail"); //创建TaskDetail bean类

entity.addIdProperty(); //增加id列


entity.addStringProperty("bonus"); //添加BONUS表项

在生成的时候要现在工作空间/weijianzhi/src手动创建(如果已经存在就需要了,否则会报错)

运行main方法生成之后会多出来四个类
- DaoMaster.java
- DaoSession.java
- TaskDetail.java
- TaskDetailDao.java

####核心初始化

````
public class TaskDBHelper {
	private static Context mContext;
	private static TaskDBHelper instance;

	private TaskDetailDao taskDetailDao;

	private TaskDBHelper() {
	}

	public static TaskDBHelper getInstance(Context context) {
		if (instance == null) {
			instance = new TaskDBHelper();
			if (mContext == null) {
				mContext = context;
			}
			DaoSession daoSession = WeijianzhiApplication.getDaoSession(mContext); // 数据库对象
			instance.taskDetailDao = daoSession.getTaskDetailDao();
		}
		return instance;
	}
}
````
初始化后通过TaskDBHelper.getInstance获取到tdb对象

####GreenDAO的增删查改

- **增** 将对象添加至数据库
````
/**
	 * 将对象添加至数据库
	 * @param taskDetail
	 */
	public void addTask(TaskDetail taskDetail){
		taskDetailDao.insert(taskDetail);
	}
````

- **删** 根据条件删除.删除全部等
````
	public void delTaskDetail(String tid){
		DeleteQuery<TaskDetail> bd = taskDetailDao.queryBuilder().where(Properties.Tid.eq(tid)).buildDelete();
		bd.executeDeleteWithoutDetachingEntities();
	}
````

- **改** 根据对象更新数据库
````
	public void updateTask(TaskDetail taskDetail){
		taskDetailDao.update(taskDetail);
	}
````

- **查:** 可以根据条件来查询 dao.queryRaw(where,params)
````
	/**
	 * 根据条件查询匹配的数据
	 */
	public List<TaskDetail> queryTaskDetail(String where, String... params) {
		return taskDetailDao.queryRaw(where, params);
	}
````

#####参考资料
* 1.https://github.com/greenrobot/greenDAO
* 2.http://my.oschina.net/u/1052509/blog/312338
* 3.http://glblong.blog.51cto.com/3058613/1354953

