我相信，大家在开发Android的过程中或多或少都会接触到 SQLite，然而我们再使用的时候需要做很多额外工作，编写SQL语句，查询结果更新等等。这事适用于Android的ORM框架就孕育而生了，现在主流的框架有 OrmLite、SugarORM、Active Android、Realm 与 GreenDAO。我们今天就给大家介绍一下GreenDAO！

##关于 greenDAO

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1507947-a94130ca017d8903.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

greenDAO 是一个将对象映射到 SQLite 数据库中的轻量且快速的 ORM 解决方案。
关于greenDAO的概念可以看官网  [greenDAO](http://greenrobot.org/greendao/)


###greenDAO 设计的主要目标

一个精简的库
性能最大化
内存开销最小化
易于使用的 APIs
对 Android 进行高度优化

###greenDAO 的主要优势
性能上优于同类的orm，这个官网有性能的比较 有兴趣的可以去官网看一下 我就不贴图了
greenDAO 支持 [protocol buffer(protobuf)](https://github.com/google/protobuf) 协议数据的直接存储，如果你通过 protobuf 协议与服务器交互，将不需要任何的映射。
与 ORMLite 等使用**注解**方式的 ORM 框架不同，greenDAO 使用「Code generation」的方式，这也是其性能能大幅提升的原因。

#让我们开始吧
###一. 在 Android 工程中配置「greenDao Generator」模块
1.在 .src/main 目录下新建一个与 java 同层级的「java-gen」目录，用于存放由 greenDAO 生成的 Bean、DAO、DaoMaster、DaoSession 等类。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1507947-67b6c47d139f54ac.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1507947-5c78a7bde2b615a4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2.配置 Android 工程（app）的 build.gradle，如图分别添加 [sourceSets](https://docs.gradle.org/current/dsl/org.gradle.api.tasks.SourceSet.html) 与[dependencies](https://docs.gradle.org/1.8-rc-1/userguide/dependency_management.html)。

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1507947-1c5b25fb70047a9b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>sourceSets {
        main {
            java.srcDirs = ['src/main/java', 'src/main/java-gen']
        }
    }

>compile 'de.greenrobot:greendao:1.3.7'

###二. 新建「greenDAO Generator」模块 (纯 Java 工程）
1.通过 File -> New -> New Module -> Java Library -> 填写相应的包名与类名 -> Finish.

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1507947-a34cf86fc42f6cbc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2.配置 新建的工程的中build.gradle，添加 dependencies.


![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1507947-0b5ecf1963c7bc35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>compile 'de.greenrobot:greendao-generator:1.3.1'

1.编写 **ExampleDaoGenerator** 类，**注意**： 我们的 Java 工程只有一个类，它的内容决定了「GreenDao Generator」的输出，你可以在这个类中通过对象、关系等创建数据库结构，下面我将以**注释**的形式详细讲解代码内容。

```
public class DaoExampleGenerator {    
public static void main(String[] args) throws Exception {       
 // 正如你所见的，你创建了一个用于添加实体（Entity）的模式（Schema）对象。     
  // 两个参数分别代表：数据库版本号与自动生成代码的包路径。        
Schema schema = new Schema(1, "com.example.junweiup.greendaodem");
//      当然，如果你愿意，你也可以分别指定生成的 Bean 与 DAO 类所在的目录，只要如下所示：
//      Schema schema = new Schema(1, "me.itangqi.bean");
//      schema.setDefaultJavaPackageDao("me.itangqi.dao");        
// 模式（Schema）同时也拥有两个默认的 flags，分别用来标示 entity 是否是 activie 以及是否使用 keep sections。       
 // schema2.enableActiveEntitiesByDefault();       
 // schema2.enableKeepSectionsByDefault();        
// 一旦你拥有了一个 Schema 对象后，你便可以使用它添加实体（Entities）了。       
 addNote(schema);        
// 最后我们将使用 DAOGenerator 类的 generateAll() 方法自动生成代码，此处你需要根据自己的情况更改输出目录（既之前创建的 java-gen)。       
 // 其实，输出目录的路径可以在 build.gradle 中设置，有兴趣的朋友可以自行搜索，这里就不再详解。        new DaoGenerator().generateAll(schema, "../GreenDAodemo/app/src/main/java-gen");    }    
/**     * @param schema     */   
 private static void addNote(Schema schema) {        
// 一个实体（类）就关联到数据库中的一张表，此处表名为「Note」（既类名）        
Entity note = schema.addEntity("Note");        
// 你也可以重新给表命名        
// note.setTableName("NODE");        
// greenDAO 会自动根据实体类的属性值来创建表字段，并赋予默认值        
// 接下来你便可以设置表中的字段：        note.addIdProperty();        note.addStringProperty("text").notNull();       
 // 与在 Java 中使用驼峰命名法不同，默认数据库中的命名是使用大写和下划线来分割单词的。        
// For example, a property called “creationDate” will become a database column “CREATION_DATE”.        
note.addStringProperty("comment");       
 note.addDateProperty("date");    }}
```

###三. 生成 DAO 文件（数据库）
执行 generator 工程，如一切正常，你将会在控制台看到如下日志，并且在主工程「java-gen」下会发现生成了DaoMaster、DaoSession、NoteDao、Note共4个类文件。
![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1507947-9fd3d280f698cb57.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果在此处出错，你可以依据错误日志进行排查，主要看是否**输出目录**存在？其他**配置**是否正确？等

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1507947-d7d92dd5ce0dd41f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###四. 在 Android 工程中进行数据库操作
我们要在Application中初始化

![Paste_Image.png](http://upload-images.jianshu.io/upload_images/1507947-b292056762730729.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里，我们只创建一个 NodeActivity 类，用于测试与讲解 greenDAO 的增、删、查功能。

```
private void addNote() {    \
String noteText = editText.getText().toString();   
 editText.setText("");   
 final DateFormat df = DateFormat.getDateTimeInstance(DateFormat.MEDIUM, DateFormat.MEDIUM);    
String comment = "Added on " + df.format(new Date());    
if (noteText == null || noteText.equals("")) {     
   ToastUtils.show(getApplicationContext(), "Please enter a note to add");   
 } else {   
     // 插入操作，简单到只要你创建一个 Java 对象      
  Note note = new Note(null, noteText, comment, new Date());   
     getNoteDao().insert(note);        
Log.d(TAG, "Inserted new note, ID: " + note.getId());      
  cursor.requery();    }}private void search() {  
  String noteText = editText.getText().toString();   
 editText.setText("");    if (noteText == null || noteText.equals("")) {     
   ToastUtils.show(getApplicationContext(), "Please enter a note to query");   
 } else {       
 // Query 类代表了一个可以被重复执行的查询        
Query query = getNoteDao().queryBuilder()                
.where(NoteDao.Properties.Text.eq(noteText))               
 .orderAsc(NoteDao.Properties.Date)              
  .build();        
// 查询结果以 List 返回        
List notes = query.list();       
 ToastUtils.show(getApplicationContext(), "There have " + notes.size() + " records");    }   
 // 在 QueryBuilder 类中内置两个 Flag 用于方便输出执行的 SQL 语句与传递参数的值    QueryBuilder.LOG_SQL = true;  
  QueryBuilder.LOG_VALUES = true;  
  Query<Note> query = getNoteDao().queryBuilder()            .where(NoteDao.Properties.Text.eq(noteText))           
 .build();   
 List<Note> list = query.list();    
Toast.makeText(NoteActivity.this, "text"+list.size(), Toast.LENGTH_SHORT).show();
}
```

##最后
本文 Demo 下载链接：[https://github.com/tangqi92/MyGreenDAO](https://github.com/JunWeiUp/GreenDao/)
本教程旨在介绍 greenDAO 的基本用法与配置，更高级与详细的使用，请参见 [官网](http://greendao-orm.com/)
如本文有任何问题欢迎指正。