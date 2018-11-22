# 工作内容
## 第一次修改erp： erp系统修改供方合同评审搜索不到上海潮诚电子有限公司
### 操作路径
* 问题:功能导航->流程管理-> 合同->供方合同评审->新建供方合同评审->选择供应单位->搜索上海潮诚
* 解决办法:
当时是直接找到SQLSelectDataBean这个类，找到问题出在这个地方但是没有写测试用例,虽然解决了上面的问题，但是导致了新的bug，这就是没有写测试用例导致的.
* 当时更改的地方是：
```java
String sqlValue = requestInfo.getParameter("sqlValue");
String queryolumn = requestInfo.getParameter("column");
//queryolumn+=",pmc081 ";//修改的是这个
String orderby = requestInfo.getParameter("orderby");
String sqlresource=requestInfo.getParameter("dataSource");//获取数据源
if(StringUtil.isNotNull(sqlresource) && !sqlresource.equals("null")){
```  

* 导致的问题是，其他的地方搜索出现了问题

* 最新的解决办法是:  
  待定  

## 第二次是回写tc_clc16
### 问题描述
由erp系统导入的Excel文件，直接在oracle表中插入了一堆数据，需求是叫我这边回写一个oa单号到tc_clc16字段中。
### 解决过程
根据debug后找到调用sql的地方，在哪个地方插入一条update语句，根据的条件是从上面的tc_clc17字段完成的更新
具体的分析过程是:
```sqlresource
update tc_clc_file set  tc_clc16 = tc_oaa10 where tc_clc17 = (select tc_oaa05 from tc_oaa_file where tc_oaa10 = ) //tc_oaa30 wbs编号   1286-012-001-001-024
用这个条件更新也可以，当tc_oaa05送审批号=tc_clc17送审批号时，更新tc_clc16OA单号=tc_oaa10OA单号
```
sql是错的，我只是根据这个找到思路
### 最后修改的地方是：
SzgtFterpMainServiceImp类中的xlftErp方法
具体修改的点是：  
```java
String strTCOAA30 = "select tc_oaa05 from tc_oaa_file where tc_oaa10 =  '" +
          fd_number + "'";
        ResultSet executeQuery = ds.executeQuery(strTCOAA30);
        String string = null;
        while (executeQuery.next()) {
          string = executeQuery.getString("tc_oaa05");
          System.out.println(string);
        }
        if (string != null) {
          String sqlupdateString = "update tc_clc_file set tc_clc16 = '" +
            fd_number + "'  where tc_clc17 = '" + string +
            "'";
          ds.executeUpdate(sqlupdateString);
        }
```
是在下面这行代码之前插入的
```java

        int size = fd_3347c27fe8337a.size();
        if (size < 1) break label794;
        for (int i = 0; i < size; i++) {
          String sqlmx = "insert into tc_clb_file (tc_clb01,tc_clb02,tc_clb03,tc_clb04,tc_clb05,tc_clb06) VALUES ('" +
            fd_idmxList.get(i) +
```  

### 刘武测试后通过
其他的问题暂时没有出现写这个是为了记录一下，以防下次导致其他的问题产生
