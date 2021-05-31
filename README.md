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

# 添加时间戳：
1.添加时间戳的位置在主页面的每个列表项中添加，即在notelist_item.xml布局文件中添加一个

       <TextView  
       android:id="@android:id/text2"  
       android:layout_width="match_parent"  
       android:layout_height="wrap_content"  
       android:textAppearance="?android:attr/textAppearanceLarge"  
       android:gravity="center_vertical"  
       android:paddingLeft="5dp"  
       android:singleLine="true" />  
       
2.在NoteList类的PROJECTION中添加COLUMN_NAME_MODIFICATION_DATE字段(该字段在NotePad中有说明)

      // The columns needed by the cursor adapter
      private static final String[] PROJECTION = new String[] {    
      NotePad.Notes._ID, // 0    
      NotePad.Notes.COLUMN_NAME_TITLE, // 1    
      NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//在这里加入了修改时间的显示    
      };    

3.修改适配器内容，增加dataColumns中装配到ListView的内容，所以要同时增加一个ID标识来存放该时间参数。

     // The names of the cursor columns to display in the view, initialized to the title column
     String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE,
     NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE //增加时间参数} ;
     // The view IDs that will display the cursor columns, initialized to the TextView in noteslist_item.xml
     int[] viewIDs = { android.R.id.text1 ,android.R.id.text2};

4.在NoteEditor文件的updateNote方法中获取当前系统的时间，并对时间进行格式化

       // Sets up a map to contain values to be updated in the provider.   
      ContentValues values = new ContentValues();  
      // 转化时间格式
      Long now = Long.valueOf(System.currentTimeMillis());  
      SimpleDateFormat sf = new SimpleDateFormat("yy/MM/dd HH:mm");  
      Date d = new Date(now);  
      String format = sf.format(d);  
      values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, format);

# 添加搜索功能：
1.搜索组件在主页面的菜单选项中，所以应在list_options_menu.xml布局文件中添加搜索功能

     <item
      android:id="@+id/menu_search"
      android:icon="@android:drawable/ic_menu_search"
      android:title="@string/menu_search"
      android:showAsAction="always" />

2.新建一个查找笔记内容的布局文件note_search.xm;

    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
     <SearchView
         android:id="@+id/search_view"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         android:iconifiedByDefault="false"
         />
     <ListView
         android:id="@+id/list_view"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         />
    </LinearLayout>

3.在NoteList类中的onOptionsItemSelected方法中添加search查询的处理(跳转)


        case R.id.menu_search:  
        //查找功能  
        //startActivity(new Intent(Intent.ACTION_SEARCH, getIntent().getData()));  
          Intent intent = new Intent(this, NoteSearch.class);  
          this.startActivity(intent);  
          return true;  

4.新建一个NoteSearch类用于search功能的功能实现

    package com.example.android.notepad;
    import android.app.Activity;
    import android.content.Intent;
    import android.database.Cursor;
    import android.database.sqlite.SQLiteDatabase;
    import android.os.Bundle;
    import android.widget.ListView;
    import android.widget.SearchView;
    import android.widget.SimpleCursorAdapter;
    import android.widget.Toast;

    public class NoteSearch extends Activity implements SearchView.OnQueryTextListener
    {
        ListView listView;
        SQLiteDatabase sqLiteDatabase;
        /**
         * The columns needed by the cursor adapter
         */
        private static final String[] PROJECTION = new String[]{
                NotePad.Notes._ID, // 0
                NotePad.Notes.COLUMN_NAME_TITLE, // 1
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE//时间
    };

    public boolean onQueryTextSubmit(String query) {
        Toast.makeText(this, "您选择的是："+query, Toast.LENGTH_SHORT).show();
        return false;
    }
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_search);
        SearchView searchView = findViewById(R.id.search_view);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listView = findViewById(R.id.list_view);
        sqLiteDatabase = new NotePadProvider.DatabaseHelper(this).getReadableDatabase();
        //设置该SearchView显示搜索按钮
        searchView.setSubmitButtonEnabled(true);

        //设置该SearchView内默认显示的提示文本
        searchView.setQueryHint("查找");
        searchView.setOnQueryTextListener(this);

    }
    public boolean onQueryTextChange(String string) {
        String selection1 = NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?";
        String[] selection2 = {"%"+string+"%","%"+string+"%"};
        Cursor cursor = sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION, // The columns to return from the query
                selection1, // The columns for the where clause
                selection2, // The values for the where clause
                null,          // don't group the rows
                null,          // don't filter by row groups
                NotePad.Notes.DEFAULT_SORT_ORDER // The sort order
        );
        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = {
                NotePad.Notes.COLUMN_NAME_TITLE,
                NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE
        } ;
        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = {
                android.R.id.text1,
                android.R.id.text2
        };
        // Creates the backing adapter for the ListView.
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                this,                             // The Context for the ListView
                R.layout.noteslist_item,         // Points to the XML for a list item
                cursor,                           // The cursor to get items from
                dataColumns,
                viewIDs
        );
        // Sets the ListView's adapter to be the cursor adapter that was just created.
        listView.setAdapter(adapter);
        return true;
    }
    }

5.最后要在清单文件AndroidManifest.xml里面注册NoteSearch,否则无法实现界面的跳转

     <activity android:name=".NoteSearch" android:label="@string/search_note" />
     
# 运行效果图

原页面

![image](https://github.com/zhanghaocreate/photo/blob/main/images/4.png)
![image](https://github.com/zhanghaocreate/photo/blob/main/images/5.png)

添加备忘录

![image](https://github.com/zhanghaocreate/photo/blob/main/images/2.png)

添加结果

![image](https://github.com/zhanghaocreate/photo/blob/main/images/1.png)

查询

![image](https://github.com/zhanghaocreate/photo/blob/main/images/3.png)


