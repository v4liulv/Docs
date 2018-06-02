# Neo4j-D3关系图学习手册

**目录 (Table of Contents)**

[TOC]

***

## 前提 

* 需要保证已经安装Neoj4j图库（详细可以查看[Neo4j操作手册](http://localhost/)）
* 开发测试数据或者正式环境数据已经入库到图库
这里不对Neo4j的数据进行说明，如果需要可以查看：
[数据建模设计](http://localhost/) , [数据入库手册](http://localhost/)
 
## 整个关系图查询展示流程

### 主界面访问
本地访问，因为公司集成环境暂时没有机器部署。
   访问地址: http://localhost:8080/neo4j/user/
   
  ![](/images/kshfx-d3-neo4j/kshfx-web01.png)  
   
### 节点查询
使用* 查询100个随机节点，或者使用姓名或者证件号码精确查询

  ![](/images/kshfx-d3-neo4j/kshfx-web02.png) 

### 查询具体的某一个人员节点的路径，可视化展示

点击"操作-关系可视化', 默认为1层路径查询
如果要修改查询多层的关系，可以修改项目的配置文件neo4j.properties中

 ![](/images/kshfx-d3-neo4j/kshfx-web03.png) 
 
 
# Neo4j模块关系图展示说明

功能：点击人员节点关联此人多层全部的关系路径查询-也就是多层递归调用查询

## 最大关系路径层数查询设置
在ne4j.properties的neo4j.kshfx.recurrent.max.num属性配置，默认为1

**注：后续版本可提供在前台Web暂时选择控制切换展示使用者想展示的层数**

## 调用递归层数设置

* js中设置
调用层数设置在/js/kshcx/d3.graph.min.js的 d3NeoQueryGraph 函数设置参数：&recurrent=1.此版本已经废除

* 控制器中设置控制
UserController类的graphByZjhmToJson方法中进行参数recurrent进行调整控制

## 数据来源

* 测试数据：/data/*.json

* Neo4j查询实时数据：d3.json()通过控制器的url传递动态传递zjhm查询Neo4j数据并组装成满足D3的Json字符串，
最终来源信息请查看Match2D3类的gernerateJsonString方法

## Neo4j连接配置

在classpath的ne4oj.properties根据安装的Neo4j实际环境进行相关配置修改

```properties
# neo4j图连接配置
# neo4j 服务器 ip 地址 （需要修改）
neo4j.kshfx.ip=192.168.0.111
# neo4j 连接用户名（需要修改）
neo4j.kshfx.username=neo4j
# neo4j 连接密码（需要修改）
neo4j.kshfx.password=huazi@123
## 连接协议可选择http或bolt,推荐使用bolt连接 （默认即可）
neo4j.kshfx.conn=bolt
## 连接Driver类 （默认即可）
neo4j.kshfx.driver=org.neo4j.ogm.drivers.bolt.driver.BoltDriver
## 如果开启debug模式，连接信息会写入neo4j Home/logs/debug.log日志中 （默认即可）
neo4j.kshfx.conn.debug=true
## kshfx的节点关系实体映射类 （默认即可）
neo4j.kshfx.domain.package=com.sinobest.kshfx.neo4j.mvc.kshcx.domain
# neo4j 查询递归调用关系最大层数,默认为1 （默认即可）
neo4j.kshfx.recurrent.max.num=1
```

## 文件和包说明

* jsp: web/views/jsp/graph_home.jsp
* js:  web/js/kshfx/ (包含d3的js)
* css: web/css/kshcx
* images: web/images/neo4j 和 web/images/users
* 控制器包：com.sinobest.kshfx.neo4j.mvc.kshcx.web.UserController
* Service包： com.sinobest.kshfx.neo4j.mvc.kshcx.service
* Repo包：com.sinobest.kshfx.neo4j.mvc.kshcx.repo
* Neo4jEntity包：com.sinobest.kshfx.neo4j.mvc.kshcx.domian


## 核心代码

核心代码注意包含Jsp、JS、Java部分

### jsp 部分
在web/view/jsp/graph_home.jsp，提供人员节点的查询以及人员关系的可视化展示

核心内容在于内置的Java代码部分接收控制器传递的Request数据进行展示：
```html
<%
System.setProperty("sun.jnu.encoding", "utf-8");
String path = request.getContextPath();
String basePath = request.getScheme() + "://" + request.getServerName() + ":" + request.getServerPort() + path;
//使用绝对地址，避免神奇的路径BUG
String name = (String) request.getAttribute("name");
if (name == null) name = "";
%>

 <%
Iterable<Userusers = (Iterable<User>) request.getAttribute("users");
if(users != null)
if(users != null) {
Iterator<UserusersIt = users.iterator();
while (usersIt.hasNext()){
User user = usersIt.next();
%>
<tr>
<td><%= user.getId()%></td>
<td><%= user.getName()%></td>
<td class='user'><%= user.getZjhm()%></td>
<td><input type="button" value="关系可视化" onclick="graphQuery('<%= user.getZjhm()%>')"/></td>
</tr>
<%
}
}
%>
```


### d3 关系图：
 **人员节点的查询**
  通过Jsp的查询按钮提供对应的查询值，进行Js调用，并且返回的相关的控制器调用
 
核心代码
**query.main.js部分**

```javascript
function search() {
    var queryValue = $("#search").find("input[name=search]").val();
    //判断字符开头如果是数字开头则判断查询为身份证号码，如果是中文字符开头则是查询姓名，如果是*开头或者是*结尾则是模糊查询姓名或者模糊查询身份证号码
    var p = /[0-9]/;
    var b = p.test(queryValue);//true,说明有数字
    //数字查询身份证号码
    if (b) {
        if (!isLike(queryValue)) {
            //精确查询
            window.location.href = path + "/neo4j/user/findByZjhm?zjhm=" + encodeURIComponent(queryValue);
        } else {
            //alert("身份证号码模糊查询");
            //去掉*
            queryValue = queryValue.substr(0, queryValue.length - 1);
            alert(queryValue);
            window.location.href = path + "/neo4j/user/findByZjhmLike?zjhm=" + encodeURIComponent(queryValue);
        }
    } else { //姓名查询
        if (isLike(queryValue)) {
            //精确查询
            //alert("姓名模糊查询");
            //去掉*
            queryValue = queryValue.substr(0, queryValue.length - 1);

            window.location.href = path + "/neo4j/user/findByNameLike?name=" + encodeURIComponent(queryValue);
        } else {
            //精确查询
            alert("姓名精确查询");
            window.location.href = path + "/neo4j/user/findByName?name=" + encodeURIComponent(queryValue);

        }
    }
    return false;
}
```

**d3.graph.min.js部分**
人员节点传递证件号码进行关系可视化展示.
具体可以查看d3.graph.min.js的函数：function d3Query(queryName) 

### neo4j 核心代码

提供Neo4j的查询并且组装数据或者SDN实体，并也组装D3满足的数据给到前端进行可视化展示，核心类为UserRepository

通过SDN或者Cypher对Neo4j进行查询和遍历等操作。

* **人员节点信息查询**
UserRepository的findByNameLike 和 findByName

```java
//通过姓名模糊查询，返回前100条的Users
@Query(value = "MATCH (u:User) " +
            "WHERE u.name =~ {name} " +
            "RETURN u limit 100")
Iterable<UserfindByNameLike(@Param("name") String name);

//通过姓名精确查询，返回姓名匹配的节点SDN实体，类似于Cypher MATCH (u:User {name={name}}) RETURN u
Iterable<UserfindByName(@Param("name") String name);
```
**注意**：此类查询执行提供到Users到Jsp界面进行表格属性类型展示

* **通过证件号码进行递归查询** 

以配置多层的节点和关系的遍历，并且转换成D3需要的Json格式数据进行可视化关系图展示。

核心代码
UserRepository的graphByZjhm提供多层查询语句，调用D3的json转换工具类进行Json数据转换

```java
/**
     * 返回Neo4j的路径查询的满足d3的Json字符串
     *
     * @param zjhm 证件号码
     * @param recurrent 递归查询的层数
     */
    default String graphByZjhm(String zjhm, int recurrent){
        recurrent = Neo4jPropertiesConfig.NEO4J_KSHFX_RECURRENT_MAX_NUM;
        String cypher = "MATCH p=((u:User {zjhm:\"" + zjhm +  "\"})-[*.." + recurrent + "]-(q)) RETURN p ";
        return Match2D3.gernerateJsonString(cypher);
    }
```

Neo4j-D3工具类Json数据转换
核心代码Match2D3类

```java
public class Match2D3 {
    private static final Logger logger = Logger.getLogger(Match2D3.class);

    public static Session getSession(){
        return Neo4jDriverSession.getInstance().getSession();
    }

    public static String gernerateJsonString(String cypher){
        Session session =  getSession();
        StatementResult result =session.run(cypher);
        StringBuilder nodeSb = new StringBuilder();
        StringBuilder relsSb = new StringBuilder();
        nodeSb.append("{\n" + "\"nodes\":[");
        relsSb.append("\"links\":[");
        Set<LongidSet = new HashSet<>();
        //d3只能按照顺序 记录node的id
        Map<Long, Longids = new HashMap<>();
        Long d3NodeId = 0L;
        while (result.hasNext()){
            Record record = result.next();
            List<Valuelist = record.values();
            for (Value v : list){
                Path path = v.asPath();
                //nodes
                for(Node n: path.nodes()){
                    Long nid = n.id();
                    if(idSet.contains(nid)) continue; //节点重复过滤
                    //id
                    idSet.add(nid);
                    nodeSb.append("\n{ ");
                    nodeSb.append("\"id\": ");
                    ids.put(nid, d3NodeId);  //ids
                    nodeSb.append(nid + " ,");
                    //lable
                    nodeSb.append("\"label\": ");
                    nodeSb.append("\"" + n.labels().iterator().next() + "\"" + " ,");
                    //prop
                    n.keys().forEach(key -{
                        nodeSb.append(" \"" + key + "\": ");
                        nodeSb.append(n.get(key) + " ,");
                    });
                    nodeSb.deleteCharAt(nodeSb.length() - 1);
                    nodeSb.append("},");
                    d3NodeId ++;  //ids value ++
                }
                //rels
                for(Relationship r : path.relationships()){
                    relsSb.append("\n{ ");
                    relsSb.append("\"source\":");
                    relsSb.append(ids.get(r.startNodeId()) + " ,");
                    relsSb.append("\"target\":");
                    relsSb.append(ids.get(r.endNodeId()) + " ,");
                    //rels prop
                    r.keys().forEach(key -{
                        relsSb.append(" \"" + key + "\": ");
                        relsSb.append(r.get(key) + " ,");
                    });
                    relsSb.append(" \"type\":\"" + r.type() +"\"");
                    relsSb.append(" },");
                }
            }
        }
        if(nodeSb.length() 2)
            nodeSb.deleteCharAt(nodeSb.length() - 1);
        nodeSb.append("\n],\n");
        if(relsSb.length() 2)
            relsSb.deleteCharAt(relsSb.length() - 1);
        relsSb.append("\n]\n" + "}");
        logger.debug("json : 开始...\n" +  nodeSb.toString() + relsSb.toString() );
        logger.debug("结束！！！");
        if(session.isOpen()){
            Neo4jDriverSession.getInstance().pushSession(session);
        }
        return (nodeSb.toString() + relsSb.toString());
    }
}
```

* **Neo4j连接类**

通过Neo4jDriver进行Neo4j数据库的连接
注意，其中需要的neo4j.properties的对Neo4j的进行适配修改，详细内容可以查看[数据入库手册](http://localhost/)

```java
 /**
     * Neo4j 连接获取Driver类，从配置文件中读取参数
     *
     * @return Neo4j Driver
     */
    private Driver initDriver(){
        String boltUrl =  "bolt://" + Config.NEO4J_IP;
        String boltUserName = Config.NEO4J_USERNAME;
        String boltPassword = Config.NEO4J_PASSWORD;
        return initDriver(boltUrl, boltUserName, boltPassword);
    }

    /**
     * 传递Neo4j的uri和user和password连接获取Driver
     *
     * @param uri Neo4j连接URI地址，新版本只能支持bolt的连接方式
     *            如：bolt://localhost
     * @param user Neo4j连接的用户名称，默认为neo4j
     * @param password Neo4连接的密码
     * @return 返回 Neo4j Driver
     */
    private Driver initDriver(String uri, String user, String password){
        return GraphDatabase.driver(uri, AuthTokens.basic(user, password));
    } 
```


               




   
   
   
  



 





 




