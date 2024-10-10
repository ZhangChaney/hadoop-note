# 007-Java-API开发HDFS

## Java HDFS API基本操作

通过Java HDFS API实现对文件系统的基本增删查改操作

```java
package com.jxyy;

import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileStatus;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;

import java.io.IOException;
import java.net.URI;
import java.net.URISyntaxException;

public class GettingStart {
    // 创建配置对象
    private Configuration conf;
    // 创建FileSystem对象
    private FileSystem fs;

    @Before
    public void connectHDFS() throws IOException, InterruptedException, URISyntaxException {
        //配置hdfs连接
        conf = new Configuration();
        // 方法一：设置文件系统操作的对象为hdfs，并配置连接地址
        // 该方法不实用uri，无法配置操作用户
        // conf.set("fs.defaultFS", "hdfs://1.95.57.18:8020/");
        // fs = FileSystem.get(conf);

        // 方法二： 该方法使用uri配置连接地址，可以配置操作用户
        URI uri = new URI("hdfs://hadoop03:8020/");
        conf.set("dfs.client.use.datanode.hostname", "true");
        fs = FileSystem.get(uri, conf, "root");
    }

    @Test
    // 创建文件夹操作, 完成后查看web页面是否创建/java-api目录
    public void mkdir() throws IOException {
        String path = "/java-api";
        Path dir = new Path(path);  // 创建/java-api目录
        // flag是结果标识符，判断操作是否成功
        boolean flag = fs.mkdirs(dir);
        if (flag) {
            System.out.println("succeed!");
        }
    }

    @Test
    // 上传文件或目录, 完成后查看web页面是否上传/hadoop.txt文件
    // 1. 准备上传文件,在src文件夹下创建data文件夹并创建hadoop.txt文件，输入任意文本内容
    public void upload() throws IOException {
        String src = "src/data/hadoop.txt";  // 本地文件路径
        String dst = "/hadoop.txt";  // hdfs上的文件路径
        // 2.通过该方法发现连接中的datanode地址为0.0.0.0:9866，因此需要先放行9866端口
        // String s = fs.getConf().get("dfs.datanode.address");
        // 3. 上传文件
        fs.copyFromLocalFile(new Path(src), new Path(dst));
        System.out.println("succeed！");
    }

    @Test
    // 移动文件或目录，完成后查看web页面是否将hadoop.txt文件移动到/java-api目录中
    public void move() throws IOException {
        String src = "/hadoop.txt";
        String dst = "/java-api/hadoop.txt";
        boolean flag = fs.rename(new Path(src), new Path(dst));
        if (flag) {
            System.out.println("succeed!");
        }
    }

    @Test
    // 重命名文件或目录，完成后查看web页面是否将/java-api/hadoop.txt改名为java.txt
    public void rename() throws IOException {
        String src = "/java-api/hadoop.txt";
        String dst = "/java-api/java.txt";
        boolean flag = fs.rename(new Path(src), new Path(dst));
        if (flag) {
            System.out.println("succeed!");
        }
    }

    @Test
    // 下载文件或目录，完成后查看本地src/data目录下是否成功下载java.txt
    public void download() throws IOException {
        String src = "/java-api/java.txt";
        String dst = "src/data/java.txt";
        fs.copyToLocalFile(new Path(src), new Path(dst));
        System.out.println("succeed!");
    }

    @Test
    // 查看目录下的所有文件
    public void listDirs() throws IOException {
        FileStatus[] files = fs.listStatus(new Path("/")); // 得到列表数组，每个元素为文件对象
        for (FileStatus file : files) {
            System.out.println(file); // 可以调用getPath()方法仅查看文件路径
        }
    }

    @Test
    // 删除文件或目录，完成后查看web页面是否将/java-api/java.txt删除
    public void delete() throws IOException {
        String src = "/java-api/java.txt";
        boolean flag = fs.delete(new Path(src), true);
        if (flag) {
            System.out.println("succeed!");
        }
    }

    @After
    public void closeHDFS() throws IOException {
        // 关闭hdfs连接
        fs.close();
    }
}

```

