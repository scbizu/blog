---
categories:
- '#伪技术相关'
- JAVA WEB
tags:
- Examination
- J2ee
title: j2ee复习
---

## J2EE复习资料






>  原稿:12届学长 区长  编辑者:13届学沫 nace








  * java的编译命令:java 和 javac





    javac demo.java
    java  demo








  * Tomcat的命令:**startup**和**shutdown**



  * Mysql基本命令:建表(Create),删表(DROP),查询(UPDATE)






#### 进入Mysql命令行界面(Console)





    mysql -h localhost -u root -p 123456(这里是写你设置的mysql密码,默认是空)






    CREATE database database_name(数据库名称)
        DROP database database_name
        //新建user用户表
        CREATE TABLE user(
            id int primary key auto_increment,
            username varchar(20) not null,
            password varchar(20) not null
         );
        //删除user用户表
        DROP table user
        //插入新的数据
        INSERT INTO user(`username`,`password`) values(`scnace`,`123456`)
        //删除user表中名字是scnace的记录
        DELETE FROM user where `username`=`scnace`
        //更改表中名字为scnace的数据为scbizu
        UPDATE user SET username='scbizu' WHERE username='scnace'
        //选取表中名字为scnace的Record
        SELECT * FROM user WHERE username=`scnace`






_Extension:_**面试**中的SQL喜欢出的题:
1. 连表查询:
   1.1. 左连接(**LEFT JOIN**):[LEFT JOIN Declaration and usecase](http://www.w3school.com.cn/sql/sql_join_left.asp)
   1.2. 右连接(**RIGHT JOIN**):[RIGHT JOIN Declaration and usecase](http://www.w3school.com.cn/sql/sql_join_right.asp)
   1.3. 全连接(**FULL JOIN**):[FULL JOIN Declaration and Usecase](http://www.w3school.com.cn/sql/sql_join_full.asp)
2. 数据库基本操作





  * 包的引入和定义import和package





        Import java.util.Scanner;
        Import java.io.*;
        Package cn.edu.zafu;//Package








  * Java控制台(cmd&&IDE)下的输入输出:







>  Scanner和System.out.print()语句






        Scanner scanner=new Scanner(System.in);
        while(scanner.hasNext()){
        //scanner.next();
        //scanner.nextInt();
        //scanner.nextBoolean();
        //scanner.nextDouble();
        //scanner.nextFloat();
           //System.out.***
        //System.out.print();
        //System.out.println();
        //System.out.printf("%d %s",(Integer),(String));
    }








  * 流程控制语句---条件语句 if switch





      if(_condition){
        //ADD YOUR CODE HERE to do under the  _condition
        }else{
        //* otherwise....
        }








    switch(_condition){
    case _condition_0:
        break;
    case _condition_1:
        break;
    default:
        break;
    }








  * 循环语句 for while





      for(int i=0;i<args.length;i++){
          //Traversal The Array
           String str=args[i];
        }
        while(i<5){
           i++;
        }








  * 变量定义 (普通变量定义及初始化 静态变量定义 常量定义)





        int i=5;
        boolean flag=true;
        float f=3.0f;
        double d=3.5;
        byte b=1;
        static final String name="nace";







>  Note:Java变量类型[final](https://en.wikipedia.org/wiki/Final_(Java))
                   [static关键字](http://beginnersbook.com/2013/04/java-static-class-block-methods-variables/)








  * 数组的定义  数组的遍历





    int a[]=int[]{1,2,3,4,5};
    for(int i=0;i <a.length;i++){
        System.out.println(a[i]);
    }








  * 字符串的使用 定义字符串 格式化字符串 字符串查找 etc.








        String str1=String.format("%s %d","nace",2);
        //Source Data to be searched
        String str2="naceoncelovedapuregirl.";
        System.out.print(str2.indexOf("S"));
        System.out.print(str2.indexOf("S",6));//Start with 1.
        System.out.print(str2.charAt(1));







>  Note:关于[String.format()](https://docs.oracle.com/javase/tutorial/java/data/numberformat.html)
       关于[String.chatAt()](http://www.tutorialspoint.com/java/java_string_charat.htm)








  * ArrayList HashSet HashMap的定义和遍历





    List<string> array=new ArrayList<String>();
    array.add("1");
    array.add("2");
    array.add("3");
    //Travelsal the ArrayList with Iterator
    for(Iterator iterator=arr.iterator();iterator.hasNext();){
    String str=(String)iterator.next();
    System.out.println(str);
    }
    for(String str:arr){
      System.out.println(str);
    }






      //HashSet
        HashSet<String> set=new HashSet<String>();
        set.add("1");
        set.add("2");
        set.add("3");
        //Travelsal the HashMap with Iterator
        for(Iterator ite =set.iterator();iterator.hasNext();){
        String str=(String)ite.next();
        System.out.println(str);
        }
        //for-each Travelsal
        for(String str:set){
          System.out.println(str);
        }

        //HashMap Code.....
        HashMap<Integer,String> map=new HashMap<Integer,String>();
        //put(index,value)  and Start with 1
        map.put(1,"1");
        map.put(2,"2");
        //Iterator Travelsal
        Set<Entry<Integer,String>> entrySet=map.entrySet();
        for(Iterator iter =entrySet.iterator();iter.hasNext();){
            Entry<Integer,String> entry=(Entry<Integer,String>) iter.next();
            System.out.println(entry.getKey()+":"+entry.getValue());
        }
        //for-each Travelsal
        for(Entry<Integer,String>entry:entrySet){
           System.out.println(entry.getKey()+":"+entry.getValue());
        }








  * JDBC连接字符串  Tomcat资源定义含义





      try{
          String url="jdbc:mysql://localhost:3306/test_nace";
          //if you wanna Unicode is utf-8
        //  String url="jdbc:mysql://localhost:3306/test_nace?useUnicode=true&characterEncoding;=Utf-8";
          String driver="com.mysql.jdbc.Driver";
          String username="root";
          String password="";//默认为空
          Class.forName(driver);
          Connection conn=DriverManager.getConnnection(url,username,password);
          System.out.println(conn);
        }catch(Exception e){
          e.printStackTrace();
        }







>  Note:[JNDI:j2ee另一种连接数据库的方式,通过设置xml](https://docs.jelastic.com/connection-to-db-via-jndi)








  * java中的排序







<blockquote>
  collection.sort()两种实现  [Comparable](https://docs.oracle.com/javase/8/docs/api/java/lang/Comparable.html)和[Comparator](https://docs.oracle.com/javase/8/docs/api/java/util/Comparator.html)
</blockquote>





        /**
          *sort.java
          */
           package practice;


    import java.util.*;
    /**
     *
     * @author My scnace
     *
     */
    public class sort {

        public static void main(String[] args) {
     //ArrayLists中的对象String 本身含有compareTo方法，所以可以直接调用sort方法，按自然顺序排序，即升序排序
            List<String> first_array=new ArrayList<String>();
            List<realclass> classarray=new ArrayList<realclass>();

            //子弹装填  loading.....
            first_array.add("a");
            first_array.add("c");
            first_array.add("b");        
            //Compare To  Collections.sort排序
            Collections.sort(first_array);
            //Shooting....
            Result res=new Result();
         //   res.outPrinter(first_array);

            //realclass real=new realclass();

            //Sort the Class
              //装填开始.....
            classarray.add(new realclass(2,"a"));
            classarray.add(new realclass(1,"c"));
            classarray.add(new realclass(3,"b"));

            Collections.sort(classarray);
          // Collections.sort(classarray,new realclass());
            //Shooting.....
            Result res2=new Result();
            res2.outclassPriter(classarray);    
        }
    }







          /**
         *realclass.java
         */
          package practice;

    import java.lang.*;
    import java.util.*;

    public class realclass implements Comparator<realclass>,Comparable<realclass>{

        private int index;

        private String foo;

        //空值构造
        realclass(){}
        //带参构造
        realclass(int index,String foo){
            this.index=index;
            this.foo=foo;
        }

        public int getIndex() {
            return index;
        }

        public void setIndex(int index) {
            this.index = index;
        }

        public String getFoo() {
            return foo;
        }

        public void setFoo(String foo) {
            this.foo = foo;
        }

        public int compareTo(realclass real){
            return (this.foo).compareTo(real.foo);
        }

        @Override
        public int compare(realclass o1, realclass o2) {
            // TODO Auto-generated method stub
            return o1.getIndex()-o2.getIndex();
        }
    }






        /**
         *result.java
         */

    package practice;

    import java.util.List;

    public class Result {

        public String outer;

        public  void outPrinter(List<String> out){
            //Tranversal
            for(String str:out){
                System.out.println(str);
            }
        }

        public void outclassPriter(List<realclass> out){
            //Tranversal
            for(realclass real:out){
                System.out.println(real.getIndex()+","+real.getFoo());
            }
        }
    }









  * 类的定义,继承(关键字 extends)







        /**
        *first.java
        */

       package practice;

    import java.util.*;
    public abstract class first {

        private int mynum;

        private String mystr;
        //含参构造
        first(int hernum,String herstr){
            this.mynum=hernum;
            this.mystr=herstr;
        }
        //不含参数构造
        first(){
            this.mynum=23;
            this.mystr=&quot;nace&quot;;
        }
        //方法
        public void Sayhellotoher(){
            System.out.println(&quot;Hi,My num is&quot;+this.mynum);
        }
        //虚函数
        public abstract void Saymyname();

    }






        /**
        *second.java
        */
    package practice;

    public class second extends first implements inter{


        public second() {
            // TODO Auto-generated constructor stub
               //构造父类
            super();        
        }

         second(int mynum,String mystr){
             //含参构造
             super(mynum,mystr);
         }
        @Override
        public void Saymyname() {
            // TODO Auto-generated method stub
            super.Sayhellotoher();
        }
      @Override
       public void howto(){
           System.out.println(&quot;I do not kown&quot;);
       }
    }







    package practice;

    /**
     * rooter.java
     * @author My Protoss
     *
     */
    public class rooter {

        public static void main(String[] args) {
            // TODO Auto-generated method stub
            second se=new second();
            se.Saymyname();
            se.howto();
        }

    }







    /**
     *inter.java
     */
    package practice;

    public interface inter {

        public default   void howto(){
            System.out.println(&quot;Hi Nace&quot;);
        }
    }








  * session的使用





## 创建和声明Session





  HttpSession mysession=request,getSession();






<blockquote>

>
> Note:The `getSession()` method should be called before anything is written to the response stream.
</blockquote>





## 测试Session是否创建成功





    String mySessionID = mySession.getId();






## 在session中绑定数据





  mysession.setAttribute(&quot;key&quot;,value);






## 获得session数据





    mysession.getAttribute(&quot;key&quot;);//return an object

    mysession.getAttributeNames(); //return an array of sessions







## 解绑session  移除





    mysession.removesession(&quot;key&quot;);








  * Struts2





[Struts2 Manual](https://cwiki.apache.org/confluence/display/WW/struts.xml+Examples)
