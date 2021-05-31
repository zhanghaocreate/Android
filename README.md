# NotePad
导入老师的框架，就可以实现一些基础功能
1.添加备忘录
2.删除备忘录

后面我通过查询资料和教程学习后，添加了时间戳和查询功能


主要的类:
NotesList类 应用程序的入口，笔记本的首页面会显示笔记的列表
NoteEditor类 编辑笔记内容的Activity
TitleEditor类 编辑笔记标题的Activity
NoteSearch类 编辑查询笔记内容的Activity
NotePadProvider 这是笔记本应用的ContentProvider，也是整个应用的关键所在

主要的布局文件：
note_editor.xml 笔记主页面布局
note_search.xml 笔记内容查询布局
notelist_item.xml 笔记主页面每个列表项布局
title_editor.xml 修改笔记主题布局

主要的菜单文件：
editor_options_menu.xml 编辑笔记内容的菜单布局
list_context_menu.xml 笔记内容编辑上下文菜单布局
list_options_menu.xml 笔记主页面可选菜单布局

数据装配：
NoteList使用SimpleCursorAdapter来装配数据，首先查询数据库的内容

 Cursor cursor = managedQuery(
         getIntent().getData(),           
         PROJECTION,                      
         null,                             
         null,                             
         NotePad.Notes.DEFAULT_SORT_ORDER);
然后通过SimpleCursorAdapter来进行装配
 SimpleCursorAdapter adapter
         = new SimpleCursorAdapter(
                   this,                             
                   R.layout.noteslist_item,          
                   cursor,                           
                   dataColumns,
                   viewIDs);
页面跳转：
不管是可选菜单、上下文菜单中的操作，还是单击列表中的笔记条目，其相应的页面跳转都是通过Intent的Action+URI进行的

添加时间戳：
1.添加时间戳的位置在主页面的每个列表项中添加，即在notelist_item.xml布局文件中添加一个

   <TextView  
       android:id="@android:id/text2"  
       android:layout_width="match_parent"  
       android:layout_height="wrap_content"  
       android:textAppearance="?android:attr/textAppearanceLarge"  
       android:gravity="center_vertical"  
       android:paddingLeft="5dp"  
       android:singleLine="true" />  `
1
2
3
4
5
6
7
8
2.在NoteList类的PROJECTION中添加COLUMN_NAME_MODIFICATION_DATE字段(该字段在NotePad中有说明)

  // The columns needed by the cursor adapter
  private static final String[] PROJECTION = new String[] {    
      NotePad.Notes._ID, // 0    
      NotePad.Notes.COLUMN_NAME_TITLE, // 1    
      NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示    
   };    
1
2
3
4
5
6
3.修改适配器内容，增加dataColumns中装配到ListView的内容，所以要同时增加一个ID标识来存放该时间参数。

  // The names of the cursor columns to display in the view, initialized to the title column
  String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,
       NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE //增加时间参数} ;
  // The view IDs that will display the cursor columns, initialized to the TextView in noteslist_item.xml
  int[] viewIDs = { android.R.id.text1 ,android.R.id.text2};
1
2
3
4
5
4.在NoteEditor文件的updateNote方法中获取当前系统的时间，并对时间进行格式化

   // Sets up a map to contain values to be updated in the provider.   
      ContentValues values = new ContentValues();  
   // 转化时间格式
      Long now = Long.valueOf(System.currentTimeMillis());  
      SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");  
      Date d = new Date(now);  
      String format = sf.format(d);  
      values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);

添加搜索功能：




原页面

![image](https://github.com/zhanghaocreate/photo/blob/main/images/4.png)
![image](https://github.com/zhanghaocreate/photo/blob/main/images/5.png)

添加备忘录

![image](https://github.com/zhanghaocreate/photo/blob/main/images/2.png)

添加结果

![image](https://github.com/zhanghaocreate/photo/blob/main/images/1.png)

查询

![image](https://github.com/zhanghaocreate/photo/blob/main/images/3.png)


