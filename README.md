## Android期中-NotePad时间戳的添加和笔记查询

### 1. 时间戳添加

**1.1首先修改noteslist_item.xml文件，给时间的显现预留空间**

观察noteslist_list.xml源代码：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">

<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeight"
    android:textAppearance="?android:attr/textAppearanceLarge"
    android:gravity="center_vertical"
    android:paddingLeft="5dip"
    android:singleLine="true"
    />
</LinearLayout>
```

结合程序运行的结果，上面的TextView做到了显示每一笔记的标题的功能。所以，同理我们再加入一个TextView控制他的大小位置让它在标题之下显示时间，修改后的noteslist_list.xml代码为：

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">

<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@android:id/text1"
    android:layout_width="match_parent"
    android:layout_height="?android:attr/listPreferredItemHeight"
    android:textAppearance="?android:attr/textAppearanceLarge"
    android:gravity="center_vertical"
    android:paddingLeft="5dip"
    android:singleLine="true"
    />
    
    <!--添加时间戳显示空间-->
    <TextView
        android:id="@+id/modifytime"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:paddingLeft="5dip"
        android:singleLine="true"
        android:gravity="center_vertical"
        />
</LinearLayout>
```

注释下面的既是我们显示时间戳的TextView

### 1.2修改时间戳显示形式

其实在原有的笔记应用中便有创建修改时间，我们可以直接拿来显示。只不过该时间戳是Long类型的，不符合普通群众的阅读习惯。所以我们要修改时间戳的类型将它转化为我们平常见到的时间表示。首先我们要知道哪里产生了修改时间，然后直接在时间产生的地方做修改，最后再将它展示在客户端。

产生时间戳是在NoteEditor类中的updateNote方法中，原本的代码为：

```java
private final void updateNote(String text, String title) {

        // Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,                   System.currentTimeMillis());
```

其中**NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE**便是修改时间对应的变量，**System.currentTimeMillis()**该方法便是产生修改时间。

还是在updateNote方法中我们利用Date类生成当前时间，并利用SimpleDateFormat将当前时间转化为**yyyy.MM.dd HH:mm:ss**类型的时间显示。修改后的updateNote方法为：

```java
private final void updateNote(String text, String title) {
        //获取当前时间
        Date a=new Date();
        //转化时间类型
        SimpleDateFormat f=new SimpleDateFormat("yyyy.MM.dd HH:mm:ss");

        // Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, f.format(a));
```

### 1.3修改NoteList

首先通过分析NotePadProvider类我们看到源代码创建数据据库的时候创建了修改时间的列，代码如下：

```java
@Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + "INTEGER"  //修改时间的列
                   + ");");
       }
```

通过注释可以看到修改时间规定整型我们改为varchar型

```java
NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + "varchar"
```

再回到NotesList类，它从数据库只投影了_ID和笔记标题两列,我们添加修改时间的投影：

```java
private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            //添加修改时间列！！！
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
    };
```

最后是将修改时间显示再UI界面上，只需要再dataColumns中添加代表修改时间的变量，在viewIDs中设定时间显示的TextView即可原本代码如下：

```java
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE } ;
int[] viewIDs = { android.R.id.text1 };
```

修改后：

```java
String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE ,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
int[] viewIDs = { android.R.id.text1, R.id.modifytime };
```

然后运行app。

### 1.4效果展示

![mage texxt](https://github.com/IYuanM/MyUI/blob/main/pictures/1.1.PNG)<br>



## 2.添加笔记查询功能

### 2.1添加Search活动

新建Search类执行查询功能，它的布局文件是search.xml

search.xml代码为：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    <!--定义搜索框-->
    <SearchView
        android:id="@+id/search_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:iconifiedByDefault="false"
        android:queryHint="搜索">
    </SearchView>
    <!--定义搜索条目的展示-->
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >
    </ListView>
</LinearLayout>
```

在Search类中，我们需要取到search.xml的ListView和SearchView，并且SearchView添加监听，使得SearchView中的文本发生变化后，就执行一次查询；为ListView添加监听，使得查询出来的每个item在被点击后，能够查看笔记的内容，代码如下:

```java
package com.example.android.notepad;

import android.app.Activity;
import android.content.ContentUris;
import android.content.Intent;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;
import android.view.Window;
import android.view.WindowManager;
import android.widget.AdapterView;
import android.widget.ListView;
import android.widget.SearchView;
import android.widget.SimpleCursorAdapter;

public class Search extends Activity implements SearchView.OnQueryTextListener{
    ListView listview;
    SQLiteDatabase sqLiteDatabase;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, 
            NotePad.Notes.COLUMN_NAME_TITLE, 
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE // 修改时间
    };
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        requestWindowFeature(Window.FEATURE_NO_TITLE);
        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);
        super.setContentView(R.layout.search);
        Intent intent = getIntent();
        if (intent.getData() == null) {
            intent.setData(NotePad.Notes.CONTENT_URI);
        }
        listview= (ListView) findViewById(R.id.list_view);//获取listview
        sqLiteDatabase=new NotePadProvider.DatabaseHelper(this).getReadableDatabase();//对数据库进行操作
        SearchView search= (SearchView) findViewById(R.id.search_view);
        search.setOnQueryTextListener(Search.this);
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
    public boolean onQueryTextChange(String newText) {//通过标题或者内容进行查询
        Cursor cursor=sqLiteDatabase.query(
                NotePad.Notes.TABLE_NAME,
                PROJECTION,
                NotePad.Notes.COLUMN_NAME_TITLE+" like ? or "+NotePad.Notes.COLUMN_NAME_NOTE+" like ?",
                new String[]{"%"+newText+"%","%"+newText+"%"},
                null,
                null,
                NotePad.Notes.DEFAULT_SORT_ORDER);
        int[] viewIDs = { android.R.id.text1,android.R.id.text2};
        SimpleCursorAdapter adapter
                = new SimpleCursorAdapter(
                Search.this,
                R.layout.noteslist_item,
                cursor,
                new String[]{NotePad.Notes.COLUMN_NAME_TITLE,NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE},
                viewIDs
        );
        listview.setAdapter(adapter);
        return true;
    }
}

```

### 2.2添加查询图标

类比于写日记的图标和复制日记的图标们创建查询日记的图标，首先找到图标编辑的xml文件——list_option_menu.xml，添加一个搜索图标：

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <!--  This is our one standard application action (creating a new note). -->
    <!--创建日记图标-->
    <item android:id="@+id/menu_add"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_add"
          android:alphabeticShortcut='a'
          android:showAsAction="always" />
    <!--  If there is currently data in the clipboard, this adds a PASTE menu item to the menu
          so that the user can paste in the data.. -->
    <!--复制图标-->
    <item android:id="@+id/menu_paste"
          android:icon="@drawable/ic_menu_compose"
          android:title="@string/menu_paste"
          android:alphabeticShortcut='p' />
    <!--搜索日记图标-->
    <item
        android:id="@+id/menu_search"
        android:icon="@android:drawable/ic_search_category_default"
        android:showAsAction="always"
        android:title="search">
    </item>
</menu>
```

然后我们要为该图标添加点击事件，在NotesList类中的onOptionsItemSelected方法中添加case：

```java
case R.id.menu_search:
                Intent intent = new Intent();
                intent.setClass(this, Search.class);
                this.startActivity(intent);
                return true;
```

### 2.3添加Search Activity类到AndroidMainfest.xml中

```xml
<activity android:name="Search" android:label="@string/title_notes_list">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```

执行app

### 2.4运行效果

![mage texxt](https://github.com/IYuanM/MyUI/blob/main/pictures/1.1.PNG)<br>
![mage texxt](https://github.com/IYuanM/MyUI/blob/main/pictures/1.1.PNG)<br>

## 附加功能



### 界面美化

#### 修改界面背景

使用图片作为界面背景，直接修改notelist_item,使用background参数修改背景，原线性布局代码：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="100dp"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
```

添加background后：

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="100dp"
    android:background="@drawable/beijing"
    android:orientation="vertical"
    android:paddingLeft="6dip"
    android:paddingRight="6dip"
    android:paddingBottom="3dip">
```

#### 效果图


![mage texxt](https://github.com/IYuanM/MyUI/blob/main/pictures/1.1.PNG)<br>


#### 写日记界面背景的修改

首先获取当前背景：

```java
        //获取当前背景
        private View view =findViewById(R.id.note);
```



然后在NoteEditor的onCreateOptionsMenu方法中添加修改背景的子菜单，并提供背景颜色选择，原代码：

```java
public boolean onCreateOptionsMenu(Menu menu) {
        // Inflate menu from XML resource
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.editor_options_menu, menu);

        // Only add extra menu items for a saved note 
        if (mState == STATE_EDIT) {
            // Append to the
            // menu items for any other activities that can do stuff with it
            // as well.  This does a query on the system for any activities that
            // implement the ALTERNATIVE_ACTION for our data, adding a menu item
            // for each one that is found.
            Intent intent = new Intent(null, mUri);
            intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
            menu.addIntentOptions(Menu.CATEGORY_ALTERNATIVE, 0, 0,
                    new ComponentName(this, NoteEditor.class), null, intent, 0, null);
        }

        return super.onCreateOptionsMenu(menu);
    }
```

修改后：

```java
  public boolean onCreateOptionsMenu(Menu menu) {
        //添加改变背景颜色的子菜单
        SubMenu colormenu=menu.addSubMenu("背景颜色");
        colormenu.setHeaderTitle("选择背景颜色");
        colormenu.add(0,FONT_RED,0,"红色");
        colormenu.add(0,FONT_BLUE,0,"蓝色");
        colormenu.add(0,FONT_GREEN,0,"绿色");
        // Inflate menu from XML resource
        MenuInflater inflater = getMenuInflater();
        inflater.inflate(R.menu.editor_options_menu, menu);

        // Only add extra menu items for a saved note 
        if (mState == STATE_EDIT) {
            // Append to the
            // menu items for any other activities that can do stuff with it
            // as well.  This does a query on the system for any activities that
            // implement the ALTERNATIVE_ACTION for our data, adding a menu item
            // for each one that is found.
            Intent intent = new Intent(null, mUri);
            intent.addCategory(Intent.CATEGORY_ALTERNATIVE);
            menu.addIntentOptions(Menu.CATEGORY_ALTERNATIVE, 0, 0,
                    new ComponentName(this, NoteEditor.class), null, intent, 0, null);
        }

        return super.onCreateOptionsMenu(menu);
    }
```

最后在onOptionsItemSelected方法中绑定对应的事件，原代码：

```java
public boolean onOptionsItemSelected(MenuItem item) {
        // Handle all of the possible menu actions.
        switch (item.getItemId()) {
        case R.id.menu_save:
            String text = mText.getText().toString();
            updateNote(text, null);
            finish();
            break;
        case R.id.menu_delete:
            deleteNote();
            finish();
            break;
        case R.id.menu_revert:
            cancelNote();
            break;
        }
        return super.onOptionsItemSelected(item);
    }
```

修改后：

```java
public boolean onOptionsItemSelected(MenuItem item) {
        View view =findViewById(R.id.note);
        // Handle all of the possible menu actions.
        switch (item.getItemId()) {
        case R.id.menu_save:
            String text = mText.getText().toString();
            updateNote(text, null);
            finish();
            break;
        case R.id.menu_delete:
            deleteNote();
            finish();
            break;
        case R.id.menu_revert:
            cancelNote();
            break;
        //添加背景修改事件
        case FONT_RED:
            view.setBackgroundColor(Color.RED);
            break;
        case FONT_BLUE:
            view.setBackgroundColor(Color.BLUE);
            break;
        case FONT_GREEN:
            view.setBackgroundColor(Color.GREEN);
            break;
        }
        return super.onOptionsItemSelected(item);
    }
```

#### 效果图

![mage texxt](https://github.com/IYuanM/MyUI/blob/main/pictures/1.1.PNG)<br>







