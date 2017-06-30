# SSM学生信息管理系统  



## 章节④   学生模块开发

### 22增加学生（数据层实现）

1. 定义vo类,名称为Student.java

   ```java
   public class Classes implements Serializable {
       private Integer cid;
       private String cname;
       private String note;
       private List<Student> students; //一个班级有多个学生

       public List<Student> getStudents() {
           return students;
       }

       public void setStudents(List<Student> students) {
           this.students = students;
       }

       public Integer getCid() {
           return cid;
       }

       public void setCid(Integer cid) {
           this.cid = cid;
       }

       public String getCname() {
           return cname;
       }

       public void setCname(String cname) {
           this.cname = cname;
       }

       public String getNote() {
           return note;
       }

       public void setNote(String note) {
           this.note = note;
       }

       @Override
       public String toString() {
           return "Classes{" +
                   "cid=" + cid +
                   ", cname='" + cname + '\'' +
                   ", note='" + note + '\'' +
                   '}';
       }
   }
   ```

   ```java
   public class Student implements Serializable {
       private String sid;
       private String name;
       private Integer age;
       private Integer sex;
       private String address;
       private Classes classes; //一个学生属于一个班级

       public String getSid() {
           return sid;
       }

       public void setSid(String sid) {
           this.sid = sid;
       }

       public String getName() {
           return name;
       }

       public void setName(String name) {
           this.name = name;
       }

       public Integer getAge() {
           return age;
       }

       public void setAge(Integer age) {
           this.age = age;
       }

       public Integer getSex() {
           return sex;
       }

       public void setSex(Integer sex) {
           this.sex = sex;
       }

       public String getAddress() {
           return address;
       }

       public void setAddress(String address) {
           this.address = address;
       }

       public Classes getClasses() {
           return classes;
       }

       public void setClasses(Classes classes) {
           this.classes = classes;
       }
   }
   ```

2. 编写studentMapper.xml

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
           "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="StudentNS">
      <!-- 实现数据增加操作 -->
       <insert id="doCreate" parameterType="Student">
           INSERT INTO student(sid,name,age,sex,address,cid) VALUES (#{sid},#{name},#{age},#{sex},#{address},#{classes.cid})
       </insert>
   </mapper>
   ```

3. 编写mybatis.cfg.xml文件

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <!DOCTYPE configuration   
       PUBLIC "-//mybatis.org//DTD Config 3.0//EN"   
       "http://mybatis.org/dtd/mybatis-3-config.dtd">
   <configuration>
       <!--配置别名-->
      <typeAliases>
           <typeAlias type="cn.ylcto.student.vo.Admin" alias="Admin"/>
           <typeAlias type="cn.ylcto.student.vo.Classes" alias="Classes"/>
           <typeAlias type="cn.ylcto.student.vo.Student" alias="Student"/>
       </typeAliases>
       <mappers>
           <mapper resource="mapper/adminMapper.xml"/>
           <mapper resource="mapper/classesMapper.xml"/>
           <mapper resource="mapper/studentMapper.xml"/>
       </mappers>
   </configuration>
   ```

4. 编写DAO接口

   ```java
   package cn.ylcto.student.dao;

   import cn.ylcto.student.dao.IDAO;
   import cn.ylcto.student.vo.Student;

   /**
    * Created by kangkang on 2017/6/29.
    */
   public interface IStudentDAO extends IDAO<String,Student> {
   }
   ```

5. 编写IStudentDAO的实现类

   ```java
   package cn.ylcto.student.dao.impl;

   import cn.ylcto.student.dao.IStudentDAO;
   import cn.ylcto.student.vo.Student;
   import org.apache.ibatis.session.SqlSessionFactory;
   import org.mybatis.spring.support.SqlSessionDaoSupport;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Repository;

   import java.sql.SQLException;
   import java.util.List;
   import java.util.Set;

   /**
    * Created by kangkang on 2017/6/29.
    */
   @Repository
   public class StudentDAOImpl extends SqlSessionDaoSupport implements IStudentDAO {
       @Autowired
       public StudentDAOImpl(SqlSessionFactory sqlSessionFactory){
           super.setSqlSessionFactory(sqlSessionFactory);
       }

       @Override
       public boolean doCreate(Student vo) throws SQLException {
           return super.getSqlSession().insert("StudentNS.doCreate",vo)>0;
       }
   }
   ```

### 23增加学生（服务层实现与JUNIT测试）

1. 定义服务层接口

   ```java
   package cn.ylcto.student.service;

   import cn.ylcto.student.vo.Student;

   /**
    * Created by kangkang on 2017/6/30.
    */
   public interface IStudentService {
       /**
        * 实现学生数据增加操作
        * @param vo 表示要执行数据增加的vo对象
        * @return 成功返回true，失败返回false
        * @throws Exception
        */
       public boolean insert(Student vo) throws Exception;
   }
   ```

2. 实现服务层接口总的增加学生的方法

   ```java
   package cn.ylcto.student.service.impl;

   import cn.ylcto.student.dao.IStudentDAO;
   import cn.ylcto.student.service.IStudentService;
   import cn.ylcto.student.vo.Student;
   import org.springframework.stereotype.Service;

   import javax.annotation.Resource;

   /**
    * Created by kangkang on 2017/6/30.
    */
   @Service
   public class StudentServiceImpl implements IStudentService {
       @Resource
       private IStudentDAO studentDAO;
       @Override
       public boolean insert(Student vo) throws Exception {
           return this.studentDAO.doCreate(vo);
       }
   }
   ```

3. 编写JUNIT测试

   ```java
   package cn.ylcto.test;

   import cn.ylcto.student.service.IStudentService;
   import cn.ylcto.student.vo.Classes;
   import cn.ylcto.student.vo.Student;
   import junit.framework.TestCase;
   import org.junit.Test;
   import org.springframework.context.ApplicationContext;
   import org.springframework.context.support.ClassPathXmlApplicationContext;


   public class StudentServiceTest {
       private static ApplicationContext ctx ;
       private static IStudentService studentService;

       static {
           ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
           studentService = ctx.getBean("studentServiceImpl",IStudentService.class);
       }
       @Test
       public void insert()throws Exception{
           Student vo = new Student();
           vo.setSid("YLCTO832");
           vo.setName("张三");
           vo.setAge(23);
           vo.setSex(1); //1表示男  0表示女
           vo.setAddress("XXXX路XXXX号");
           Classes classes = new Classes();
           classes.setCid(1);
           vo.setClasses(classes);
           TestCase.assertTrue(this.studentService.insert(vo));
       }
   }
   ```

### 24增加学生（控制层实现 ）

1. 新建一个StudentAction.java类

   ```java
   package cn.ylcto.student.action;

   import cn.ylcto.student.service.IStudentService;
   import cn.ylcto.student.vo.Student;
   import cn.ylcto.util.action.DefaultAction;
   import org.springframework.stereotype.Controller;
   import org.springframework.web.bind.annotation.RequestMapping;
   import org.springframework.web.servlet.ModelAndView;

   import javax.annotation.Resource;

   /**
    * Created by kangkang on 2017/6/30.
    */
   @Controller
   @RequestMapping(value="/pages/back/student/*")
   public class StudentAction extends DefaultAction {
       @Resource
       private IStudentService studentService;

       @RequestMapping(value = "student_insert")
       public ModelAndView insert(Student vo) {
           ModelAndView mav = new ModelAndView(super.getResource("pages.forward"));
           try {
               if (this.studentService.insert(vo)){ //表示数据增加成功
                   super.setMsgAndPath(mav,"student.insert.success","student.login.success");
               }else{ //表示数据增加失败
                   super.setMsgAndPath(mav,"student.insert.failure","student.login.failure");
               }

           } catch (Exception e) {
               e.printStackTrace();
           }
           return mav;
       }

       @Override
       public String getText() {
           return "学生";
       }
   }
   ```
