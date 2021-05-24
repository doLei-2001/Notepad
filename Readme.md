# Notepad 安卓笔记本优化项目

[TOC]

# 一、项目前言

​	本项目为福建师范大学2019级软件工程（闽台合作）专业的专业实践课期末大作业。本人在林立老师的指导下，学习相关安卓移动开发知识。在老师授课和学习他人的基础上独立完成了此份作业。

​	本篇内容为大作业的第一部分：为笔记<u>加上时间戳</u>和<u>添加笔记查询</u>功能。

# 二、实验分析

​	针对为笔记加上时间戳和添加笔记查询功能的问题，简单分析来可以归结为以下步骤：

<img src="[Notepad/实验分析流程图.png at master · doLei-2001/Notepad (github.com)](https://github.com/doLei-2001/Notepad/blob/master/Resources/实验分析流程图.png)" alt="image-20210518154040923" style="zoom:67%;" />

# 三、实验步骤

## 3.1 为笔记添加时间戳

### 3.1.1 代码实现

#### 	①实现对数据表的创建。

​	首先，修改NotePadProvider中的onCreate()函数，在其中加入时间这一属性列。

```Java
@Override
       public void onCreate(SQLiteDatabase db) {
           //创建数据表
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " INTEGER"
                   + ");");
       }
```

​	接着，在NoteList中加入时间这一属性列的投影。

```Java
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//添加修改时间
    };
```

​    然后，使用Cursor进行数据库查询，之后再用Adapter进行装填。

```Java
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE }//加入修改时间;
int[] viewIDs = { android.R.id.text1, R.id.text2 }//加入修改时间;
```

#### 	②获取当前时间并整理时间戳格式

​	在NotePadProvider中的insert方法：

```java
Long now = Long.valueOf(System.currentTimeMillis());
//修改 需要将毫秒数转换为时间的形式yy.MM.dd HH:mm:ss
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String dateFormat = simpleDateFormat.format(date);//转换为yy.MM.dd HH:mm:ss形式的时间
// If the values map doesn't contain the creation date, sets the value to the current time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_CREATE_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_CREATE_DATE, dateFormat);
}

// If the values map doesn't contain the modification date, sets the value to the current
// time.
if (values.containsKey(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE) == false) {
    values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
}
```

​	在NoteEditor中的updateNote方法:

```Java
long now = System.currentTimeMillis();
Date date = new Date(now);
SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yy-MM-dd HH:mm:ss");
String dateFormat = simpleDateFormat.format(date);
values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, dateFormat);
```

#### 	③在noteslist_item.xml中加入一个新的TextView。

```xml
<RelativeLayout android:layout_height="match_parent"
    android:layout_width="match_parent"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <TextView xmlns:android="http://schemas.android.com/apk/res/android"
        android:id="@android:id/text1"
        android:layout_width="match_parent"
        android:layout_height="?android:attr/listPreferredItemHeight"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:gravity="center_vertical"
        android:paddingLeft="5dip"
        android:singleLine="true"
    />
    <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="5dip"
        android:singleLine="true"
        android:gravity="center_vertical"
    />
</RelativeLayout>
```

### 	3.1.2 结果展示

<img src="[Notepad/时间戳功能_结果展示.png at master · doLei-2001/Notepad (github.com)](https://github.com/doLei-2001/Notepad/blob/master/Resources/时间戳功能_结果展示.png)" alt="image-20210518154040923" style="zoom:67%;" />

## 3.2 笔记查询功能的插入

### 	3.2.1 代码实现

#### 	①添加搜索按钮

​	首先，在函数onCreateOptionsMenu(Menu menu)中插入代码块：

```java
MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.list_options_menu, menu);
```

​	找到list_option_menu.xml文件，添加一个搜索图标：

```xml
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <!--  This is our one standard application action (creating a new note). -->
    <item android:id="@+id/menu_add"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_add"
          android:alphabeticShortcut='a'
          android:showAsAction="always" />
    <!--  If there is currently data in the clipboard, this adds a PASTE menu item to the menu
          so that the user can paste in the data.. -->
    <item android:id="@+id/menu_paste"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_paste"
          android:alphabeticShortcut='p' />
    <!--添加一个item-->
    <item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always"
        android:title="search">
    </item>
</menu>
```

#### 	②针对搜索图标添加点击事件

​	首先，在函数boolean onOptionsItemSelected(MenuItem item)中添加case:

```java
case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(this, NoteSearch.class);
                this.startActivity(intent);
                return true;
```

#### 	③添加note_search.xml和NoteSearch活动

编辑note_search.xml:

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="搜索">
    </SearchView>
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
    </ListView>
</LinearLayout>
```

​	为实现具体内容的查询功能，我们在NoteSearch.java中需要为SearchView添加监听，使得SearchView中的文本在发生变化后，就可以执行一次查询；并且为ListView添加监听，使得查询出来的结果笔记在被点击后，可以实现跳转。代码如下：

```java
public class NoteSearch extends Activity implements SearchView.OnQueryTextListener{
    ListView listview;//
    SQLiteDatabase sqLiteDatabase;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 修改时间
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);//设置全屏显示
        super.setContentView(R.layout.note_search);
        Intent intent = getIntent();

        // If there is no data associated with the Intent, sets the data to the default URI, which
        // accesses a list of notes.
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listview= (ListView) findViewById(R.id.list_view);//获取listview
        sqLiteDatabase=new NotePadProvider.DatabaseHelper(this).getReadableDatabase();//对数据库进行操作
        SearchView search= (SearchView) findViewById(R.id.search_view);//获取搜索视图
        search.setOnQueryTextListener(NoteSearch.this);
        listview.setOnItemClickListener(new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> adapterView, View view, int i, long l) {
            //为每个item添加点击事件，点击可以查看笔记具体内容
                Uri uri = ContentUris.withAppendedId(getIntent().getData(), l);
                String action = getIntent().getAction();
                if (Intent.ACTION_PICK.equals(action) || Intent.ACTION_GET_CONTENT.equals(action)) {
                    setResult(RESULT_OK, new Intent().setData(uri));
                } else {
                    startActivity(new Intent(Intent.ACTION_EDIT, uri));
                }
            }
        });
    }

    @Override
    public boolean onQueryTextSubmit(String query) {
        return true;
    }

    @Override
    public boolean onQueryTextChange(String newText) {//实现模糊查询，通过标题或者内容进行查询
        Cursor cursor=sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION,
                NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?",
                new String[]{"%"+newText+"%","%"+newText+"%"},
                null,
                null,
                NotePad.Notes.DEFAULT_SORT_ORDER);
        int[] viewIDs = { R.id.text3,R.id.text4};
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                NoteSearch.this,                             
                R.layout.searchlist_item,          
                cursor,                           
                new String[]{NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE},
                viewIDs
        );
        listview.setAdapter(adapter);
        return true;
    }
}
```

### 3.2.2 结果展示

<img src="[Notepad/笔记查询功能_结果展示.png at master · doLei-2001/Notepad (github.com)](https://github.com/doLei-2001/Notepad/blob/master/Resources/笔记查询功能_结果展示.png)" alt="image-20210522140357279" style="zoom:67%;" />

# 四、实验总结

​	在本次的实验中，我在林立老师的指导下，通过查阅相关资料独立完成了此部分的实验。

​    在做完本次实验后，我初步了解了安卓开发的具体步骤和流程，发现无论是安卓移动开发还是web应用开发都具有相同的模式。二者本质上是类似的，只是相关知识的不同。









