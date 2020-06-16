---
title: 不使用spring的情况下用java原生代码操作mongodb数据库的两种方式
date: 2016-11-07 15:09:42
tags: [数据库,mongodb,java]
toc: true
categories: 数据库
---
由于更改了mongodb3.0数据库的密码，导致这几天storm组对数据进行处理的时候，一直在报mongodb数据库连接不上的异常。
<!--more-->
主要原因实际上是和mongodb本身无关的，因为他们改的是配置文件的密码，而实际上这个密码在代码中根本就没有使用，他们在代码中已经把用户验证信息写死。
 
在协助他们解决这个问题的时候，我看到他们代码中在和mongodb数据库交互时使用了已经不被建议使用的方法，于是便抽时间尝试了一下另一种被建议的方式实现各功能。
 
当然了，生产环境中用的是mongodb集群，验证时的写法和单机时会略有不同，我这里就只拿单机试验。
 
使用原生的Java代码操作mongodb数据库，就不需要和spring集成的那些jar包，只用到了mongodb-java-driver3.0.3.jar，代码如下，一些需要注意的地方也都写在注释中：
<pre> 
package monAndMysql;  
import java.util.ArrayList;  
import java.util.Date;  
import java.util.List;  
import java.util.Set;  
import org.bson.BsonDocument;  
import org.bson.BsonString;  
import org.bson.Document;  
import org.bson.conversions.Bson;  
import com.mongodb.BasicDBObject;  
import com.mongodb.DB;  
import com.mongodb.DBCollection;  
import com.mongodb.DBCursor;  
import com.mongodb.DBObject;  
import com.mongodb.MongoClient;  
import com.mongodb.MongoCredential;  
import com.mongodb.ServerAddress;  
import com.mongodb.client.FindIterable;  
import com.mongodb.client.MongoCollection;  
import com.mongodb.client.MongoCursor;  
import com.mongodb.client.MongoDatabase;  
import com.mongodb.client.model.Filters;  
/** 
 *mongodb和mysql性能测试  
 *@author tuzongxun123  
 */  
public class MonAndMysqlTest {  
    public static void main(String[] args) {  
       mongodbTest();  
    }  
    public static void mongodbTest() {  
       ServerAddress sa = new ServerAddress("192.168.0.7", 27017);  
       List<MongoCredential> mongoCredentialList = newArrayList<MongoCredential>();  
       // java代码连接mongodb3.0数据库验证，userName,dbName,password  
       mongoCredentialList.add(MongoCredential.createMongoCRCredential(  
              "admin", "admin", "123456".toCharArray()));  
       MongoClient client = new MongoClient(sa, mongoCredentialList);  
       // 第一种方式  
       // 第一种方式获取db，该方法已经不建议使用  
       DB mongoDB = client.getDB("mongoTest1");  
       DBCollection collection1 = mongoDB.getCollection("userTest1");  
       // 这里的数据类型是dbobject  
       DBObject document1 = new BasicDBObject();  
       document1.put("name", "mongoTest1");  
       document1.put("createTime", new Date().getTime());  
       // 插入数据  
       collection1.insert(document1);  
       // 查询数据  
       DBCursor cursor1 = collection1.find();  
       System.out.println("第一种方式插入数据的结果：");  
       while (cursor1.hasNext()) {  
           DBObject object = cursor1.next();  
           Set<String> keySet = object.keySet();  
           for (String key : keySet) {  
              System.out.println(key + ":" + object.get(key));  
           }  
       }  
       // 更改数据  
       DBObject query = new BasicDBObject();  
       DBObject update = new BasicDBObject();  
       query.put("name", "mongoTest1");  
       update.put("$set", new BasicDBObject("name", "update1"));  
       collection1.update(query, update);  
       System.out  
           .println("--------------------------------------------------------------------------------------");  
       System.out.println("第一种方式修改数据的结果：");  
       DBCursor cursor11 = collection1.find();  
       while (cursor11.hasNext()) {  
           DBObject object = cursor11.next();  
           Set<String> keySet = object.keySet();  
           for (String key : keySet) {  
              System.out.println(key + ":" + object.get(key));  
           }  
       }  
       // 删除数据  
       DBObject query1 = new BasicDBObject();  
       query1.put("name", "update1");  
       collection1.remove(query1);  
       System.out  
           .println("--------------------------------------------------------------------------------------");  
       System.out.println("第一种方式删除数据的结果：");  
       DBCursor cursor12 = collection1.find();  
       while (cursor12.hasNext()) {  
           DBObject object = cursor12.next();  
           Set<String> keySet = object.keySet();  
           for (String key : keySet) {  
              System.out.println(key + ":" + object.get(key));  
           }  
       }  
       // 第二种方式  
       System.out  
               .println("****************************************************************************");  
       // 第二种方式获取db及插入数据和查询操作。推荐方式  
       MongoDatabase database = client.getDatabase("mongoTest2");  
       // 注意这里的数据类型是document  
       Document document2 = new Document();  
       document2.put("name", "mongoTest2");  
       document2.put("createTime", new Date().getTime());  
       MongoCollection collection2 = database.getCollection("userTest2");  
       // 插入数据  
       collection2.insertOne(document2);  
       // 查询数据，注意这里直接查询出的结果不是游标，还需要转换  
       FindIterable<Document> findIterable = collection2.find();  
       MongoCursor<Document> cursor2 = findIterable.iterator();  
       System.out.println("第二种方式插入数据的结果：");  
       while (cursor2.hasNext()) {  
           Document document = cursor2.next();  
           Set<String> keySet = document.keySet();  
           for (String key : keySet) {  
              System.out.println(key + ":" + document.get(key));  
           }  
       }  
       // 更改数据  
       Bson filter = Filters.eq("name", "mongoTest2");  
       BsonDocument update2 = new BsonDocument();  
       update2.put("$set", new BsonDocument("name", new BsonString("update2")));  
       collection2.updateOne(filter, update2);  
       System.out  
           .println("--------------------------------------------------------------------------------------");  
       MongoCursor<Document> cursor21 = findIterable.iterator();  
       System.out.println("第二种方式更改数据的结果：");  
       while (cursor21.hasNext()) {  
           Document document = cursor21.next();  
           Set<String> keySet = document.keySet();  
           for (String key : keySet) {  
              System.out.println(key + ":" + document.get(key));  
           }  
       }  
       // 删除数据  
       Bson filter2 = Filters.eq("name", "update2");  
       collection2.deleteOne(filter2);  
       System.out  
           .println("--------------------------------------------------------------------------------------");  
       MongoCursor<Document> cursor22 = findIterable.iterator();  
       System.out.println("第二种方式删除数据的结果：");  
       while (cursor22.hasNext()) {  
           Document document = cursor22.next();  
           Set<String> keySet = document.keySet();  
           for (String key : keySet) {  
              System.out.println(key + ":" + document.get(key));  
           }  
       }  
       // 关闭数据库连接  
       client.close();  
    }   
}  
</pre>

执行main方法后，控制台打印结果如下图所示，证明操作都是没有问题的：
![结果](/images/mongodb/mongo1.png)