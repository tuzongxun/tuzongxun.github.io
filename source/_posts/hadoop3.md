---
title: hdfs基础操作（命令行和java代码）
date: 2020-8-10 16:59:42
tags: hadoop
toc: true
categories: hadoop
---
hadoop分布式模式初步搭建完成，无论是从命令行还是web界面都看起来是可用的，然后便可以进入下一步，可以说是进一步的验证，也可以说是hdfs相关的学习。
hdfs是分布式文件存储系统，可以进行文件的增删改查操作，原生支持的就有基本的命令行，然后就是各种语言的客户端。
这一部分，主要是记录和练习基本的操作，也当是进一步验证之前环境安装的是否可用。

<!--more-->
## 环境说明
以下内容均基于`hadoop3.1.3`版本。

## 命令行操作
### 创建目录
文件系统落到实际，自然就是目录和文件，所以首先是对文件目录的创建：
```
hdfs dfs -mkdir /test1
```
网上很多教程这里写的是`hadoop`而不是`hdfs`，可能是旧版命令，现在如果使用`hadoop`操作，虽然也可以，但是会输出提示，让替换成`hdfs`。


## 列出文件和目录列表
上边创建了目录，要进一步验证目录是否创建成功，就可以执行下边命令列出文件目录：
```
hdfs dfs -ls /
```
上边意思是列出hdfs文件系统根目录下的目录和文件列表，要再三强调的是，是hdfs文件系统，额不是执行上述命令所在系统的文件系统。
如我这里，之前已经创建过几个目录，所以执行上述命令之后就会出现如下内容：
```
Found 4 items
drwxr-xr-x   - root supergroup          0 2020-08-07 22:23 /demo1
drwxr-xr-x   - root supergroup          0 2020-08-07 01:27 /foodir
drwxr-xr-x   - root supergroup          0 2020-08-10 18:56 /hbase
drwxr-xr-x   - root supergroup          0 2020-08-10 19:15 /test1
```

### linux中文件创建
有了目录之后，就可以上传文件到hdfs，这里拓展一下linux中文件创建的简单操作，首先是`echo`，之前实际上提到过，echo可以用如下方式创建文件：
```
echo "test" >test2.txt
echo "test1" >> test3.txt
```
上述两个操作，如果文件不存在，都会创建文件并写入内容，如果文件存在，第二个会追加到文件中，第一个则会覆盖文件中的原有内容。
但是，像上边这样使用echo创建文件，必须是有内容写入，如果是`echo test2.txt`这样就不会创建文件，只会打印输出到控制台，创建空白文件需要使用`touch test2.txt`这样的操作。

### 文件上传到hdfs
有了文件后，就可以把本地文件上传到hdfs文件系统中：
```
hdfs dfs -put test3.txt /test1/
```
以上命令的意思是，把执行命令的当前目录下test3.txt文件上传到hdfs的/test1这个目录下。
这里要注意的是，如果我们再次执行上传命令，会发现不能再上传，而是报错：
```
2020-08-10 19:35:54,862 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
put: `/test1/test3.txt': File exists
```
文件已经存在，则不允许再次上传，这里其实涉及到hdfs文件存储的设计思想。
**hdfs是一个分布式文件操作系统，底层都是很多的小文件，而这些小文件是把一个大文件解析成字节数组，然后以字节拆分。**
**除了最后一个小文件，其他的小文件里字节数是一样的，然后每个小文件会有偏移量，进而使得后续的大数据量的计算性能更高。**
由于这种设计，则如果要修改一个文件，就可能涉及到各个小文件的偏移量以及字节重组和拆分，这里边就会有非常多的问题存在。
因此，hdfs的文件修改，实际只支持在文件末尾追加内容，而不允许对已有内容做修改，也正因为如此，已存在文件时就无法再上传，底层不是想象中的覆盖那么简单。

### 追加文件内容
如上所说，hdfs的文件修改实际只支持末尾追加，因为末尾追加智慧涉及到最后一个小文件的处理，而不至于可能要操作所有小文件，追加文件内容操作如下：
```
hdfs dfs -appendToFile test3.txt /test1/test3.txt
```

### 查看hdfs中某个文件内容
文件上传成功以后，可以用上边文件列表查看的方式确认是否上传成功，但是这个只能看到文件大概信息，如果要确认是否文件里的内容都已写入，则可以用查看文件内容的操作：
```
hdfs dfs -cat /test1/test3.txt
```
如我这里操作后就会输出去如下信息，最后一行就是文件里的内容：
```
2020-08-10 19:31:00,019 WARN util.NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable
2020-08-10 19:31:02,046 INFO sasl.SaslDataTransferClient: SASL encryption trust check: localHostTrusted = false, remoteHostTrusted = false
test1
```

### 删除文件
上边有了增查改，作为一个文件系统，自然也是要有删除操作的，如下：
```
hdfs dfs -rm /test1/test2.txt
```

以上便是基于hadoop自带的命令行进行hdfs的一些基础操作，再重复一句的是，我上边随便目录使用命令的前提，是要配置类hadoop相关的环境变量。
作为一个java程序员，如果学hadoop仅限于命令行，自然是不靠谱的，因此也尝试了java客户端进行了一些类似上边命令行的简单操作。
一下内容基本都是参考网上的内容，示例操作过于简单，所以没有太大改动，也没有太多说明，仅作为基础验证和记录。

## java操作

### 依赖包导入
首先是依赖包导入，我的项目虽然是springboot项目，但是似乎现在还没有官方的springboot相关的hadoop的starter，所以就要直接引入hadoop的jar，由于有些依赖冲突，所以进行了排除，最终maven配置如下：
```
<dependency>
   <groupId>org.apache.hadoop</groupId>
   <artifactId>hadoop-hdfs</artifactId>
   <version>3.1.3</version>
   <exclusions>
      <exclusion> <groupId>org.slf4j</groupId> <artifactId>slf4j-log4j12</artifactId></exclusion>
      <exclusion> <groupId>log4j</groupId> <artifactId>log4j</artifactId> </exclusion>
      <exclusion> <groupId>javax.servlet</groupId> <artifactId>servlet-api</artifactId> </exclusion>
   </exclusions>
</dependency>
<dependency>
   <groupId>org.apache.hadoop</groupId>
   <artifactId>hadoop-common</artifactId>
   <version>3.1.3</version>
   <exclusions>
      <exclusion> <groupId>org.slf4j</groupId> <artifactId>slf4j-log4j12</artifactId></exclusion>
      <exclusion> <groupId>log4j</groupId> <artifactId>log4j</artifactId> </exclusion>
      <exclusion> <groupId>javax.servlet</groupId> <artifactId>servlet-api</artifactId> </exclusion>
   </exclusions>
</dependency>
<dependency>
   <groupId>org.apache.hadoop</groupId>
   <artifactId>hadoop-client</artifactId>
   <version>3.1.3</version>
   <exclusions>
      <exclusion> <groupId>org.slf4j</groupId> <artifactId>slf4j-log4j12</artifactId></exclusion>
      <exclusion> <groupId>log4j</groupId> <artifactId>log4j</artifactId> </exclusion>
      <exclusion> <groupId>javax.servlet</groupId> <artifactId>servlet-api</artifactId> </exclusion>
   </exclusions>
</dependency>
```

### 获取hdfs文件系统对象
java操作hdfs，需要先获取到文件对象，执行url和用户名等，连接配置很多，需要实际项目需要时补充，基础可用的简单代码如下：
```
private static String hdfsPath = "hdfs://192.168.139.9:9000";
/**
 * 获取HDFS文件系统对象
 *
 * @return
 * @throws Exception
 */
private static FileSystem getFileSystem() throws Exception
{
    FileSystem fileSystem = FileSystem.get(new URI(hdfsPath), getConfiguration(), "root");
    return fileSystem;
}
/**
 * 获取HDFS配置信息
 *
 * @return
 */
private static Configuration getConfiguration() {
    Configuration configuration = new Configuration();
    configuration.set("fs.defaultFS", hdfsPath);
    return configuration;
}
```

### java中hdfs基础的增删改查
连接上hdfs，获取到文件系统对象后，就可以进行相关的操作，如下所示：
```
/**
 * 在HDFS创建文件夹
 *
 * @param path
 * @return
 * @throws Exception
 */
public static void mkdir(String path) throws Exception
{
    FileSystem fs = getFileSystem();
    // 目标路径
    Path srcPath = new Path(path);
    boolean isOk = fs.mkdirs(srcPath);
    fs.close();
}
/**
 * 读取HDFS目录信息
 *
 * @param path
 * @return
 * @throws Exception
 */
public static void readPathInfo(String path)
    throws Exception
{
    FileSystem fs = getFileSystem();
    // 目标路径
    Path newPath = new Path(path);
    FileStatus[] statusList = fs.listStatus(newPath);
    List<Map<String, Object>> list = new ArrayList<>();
    if (null != statusList && statusList.length > 0) {
        for (FileStatus fileStatus : statusList) {
            System.out.print("filePath:"+fileStatus.getPath());
            System.out.println(",fileStatus:"+ fileStatus.toString());
        }
    }
}
/**
 * HDFS创建文件
 *
 * @throws Exception
 */
public static void createFile()
    throws Exception
{
    File myFile = new File("C:\\Users\\tuzongxun\\Desktop\\tzx.txt");
    FileInputStream fis = new FileInputStream(myFile);
    String fileName = myFile.getName();
    FileSystem fs = getFileSystem();
    // 上传时默认当前目录，后面自动拼接文件的目录
    Path newPath = new Path("/demo1/" + fileName);
    // 打开一个输出流
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    byte[] b = new byte[1024];
    int n;
    while ((n = fis.read(b)) != -1) {
        bos.write(b, 0, n);
    }
    fis.close();
    bos.close();
    FSDataOutputStream outputStream = fs.create(newPath);
    outputStream.write(bos.toByteArray());
    outputStream.close();
    fs.close();
}
/**
 * 读取文件列表
 * @param path
 * @throws Exception
 */
public static void listFile(String path)
    throws Exception
{
    FileSystem fs = getFileSystem();
    // 目标路径
    Path srcPath = new Path(path);
    // 递归找到所有文件
    RemoteIterator<LocatedFileStatus> filesList = fs.listFiles(srcPath, true);
    while (filesList.hasNext()) {
        LocatedFileStatus next = filesList.next();
        String fileName = next.getPath().getName();
        Path filePath = next.getPath();
        System.out.println("##########################fileName:" + fileName);
        System.out.println("##########################filePath:" + filePath.toString());
    }
    fs.close();
}
/**
 * 读取HDFS文件内容
 *
 * @param path
 * @return
 * @throws Exception
 */
public static String readFile(String path) throws Exception
{
    FileSystem fs = getFileSystem();
    // 目标路径
    Path srcPath = new Path(path);
    FSDataInputStream inputStream = null;
    try {
        inputStream = fs.open(srcPath);
        // 防止中文乱码
        BufferedReader reader = new BufferedReader(new InputStreamReader(inputStream));
        String lineTxt = "";
        StringBuffer sb = new StringBuffer();
        while ((lineTxt = reader.readLine()) != null) {
            sb.append(lineTxt);
        }
        return sb.toString();
    }
    finally {
        inputStream.close();
        fs.close();
    }
}
/**
 * 上传HDFS文件
 *
 * @param path
 * @param uploadPath
 * @throws Exception
 */
public static void uploadFile(String path, String uploadPath) throws Exception
{
    if (StringUtils.isEmpty(path) || StringUtils.isEmpty(uploadPath)) {
        return;
    }
    FileSystem fs = getFileSystem();
    // 上传路径
    Path clientPath = new Path(path);
    // 目标路径
    Path serverPath = new Path(uploadPath);

    // 调用文件系统的文件复制方法，第一个参数是否删除原文件true为删除，默认为false
    fs.copyFromLocalFile(false, clientPath, serverPath);
    fs.close();
}
/**
 * 调用
 * @param args
 */
public static void main(String[] args) {
    try {
        //创建目录
        //mkdir("/test2");
        //列出目录列表
        readPathInfo("/");
        //列出文件列表
        // listFile("/");
        // 创建文件
        //  createFile();
        // 读取文件内容
         String a = readFile("/test/test2.txt");
        // System.out.println("###########################" + a);
        //上传文件
        //uploadFile("C:\\Users\\tuzongxun\\Desktop\\tzx.txt", "/test2");
    }
    catch (Exception e) {
        e.printStackTrace();
    }
}
```

注：以上依赖包也可以改为springboot版，同时需要额外引入common-lang3相关jar：
```
<dependency>
	<groupId>org.springframework.data</groupId>
	<artifactId>spring-data-hadoop</artifactId>
	<version>2.5.0.RELEASE</version>
</dependency>
```