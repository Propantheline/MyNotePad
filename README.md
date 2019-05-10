# MyNotePad
##  一.实验

**基于NotePad笔记本应用的修改**

## 二.实验要求

- 基本功能

   1. NoteList中显示条目增加时间戳显示

   2. 添加笔记查询功能（根据标题查询）

      

- 附加功能
  	1. UI美化
   	2. 文本字体大小颜色修改
   	3. 背景更换

## 三.实验步骤

###  项目工程结构

 ![项目工程结构图](https://img-blog.csdnimg.cn/20190509163920161.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTMxNTI5NA==,size_16,color_FFFFFF,t_70 "项目工程结构")
### 重点文件介绍
 		1. ``NotePad.java ``: 契约类，内容主要是用来声名Static常量的类；
 		2. ``NoteEditor.java``: 主要包含有修改记事条目标题，菜单按钮的响应函数，显示标签时间的函数，附加的背景更换函数定义在此；
 		3. ``NotePadProvider.java``: 继承于ContentProvider类，主要包含有数据库SQLLite的操作函数，以及数据库表的信息等；
 		4. ``TitleEditor.java``: 可以编辑标题，以及自动添加标题的实现；
 		5. ``NoteList.java``:显示在主界面上的每条笔记的内容；
 		6. ``NoteColor``:用来实现附加的更换背景功能；
 		7. `` listview.xml``:通过listview实现动态添加笔记内容
 		8. ``note_color.xml``:实现更换背景的图片按钮
 		9. ``note_editor.xml``:用来实现NoteEditor的布局文件
 		10. ``notelist_item.xml``:每条记事本标签内容的布局文件
 		11. ``title_editor.xml``:弹出一个编辑标题的对话框布局
 	

### 各部分功能的实现	   
-  基础功能---**时间戳的实现**
 1.  在notelist_item.xml中添加Textview显示时间戳
 ```
  <TextView
        android:id="@+id/text2"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:textAppearance="?android:attr/textAppearanceLarge"
        android:textSize="12dp"
        android:gravity="center_vertical"
        android:paddingLeft="10dip"
        android:singleLine="true"
        android:layout_weight="1"
        android:layout_margin="0dp"
        />
```		
2. 在数据库中添加相应的时间字段
```
@Override
       public void onCreate(SQLiteDatabase db) {
           db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                   + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                   + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                   + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                   + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " Text,"
                   + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER"//color
                   + ");");
       }
```
3. 因为显示的时间是一串数字，所以需要修改一下类型，在NoteEditor中添加SimpleDateFormat调整时间类型，并将其格式化存入数据库
```
Date nowTime = new Date(System.currentTimeMillis());
       // System.out.println(System.currentTimeMillis());
               SimpleDateFormat sdFormatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String retStrFormatNowDate = sdFormatter.format(nowTime);
        //System.out.println(retStrFormatNowDate);
        // Sets up a map to contain values to be updated in the provider.
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, retStrFormatNowDate);
 ```
 4. 最后在NotesList中增加时间字段的描述

```
   private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            //扩展  颜色
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
```
#### 显示效果
![时间戳的实现](https://img-blog.csdnimg.cn/20190509182641105.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTMxNTI5NA==,size_16,color_FFFFFF,t_70 "时间戳的实现")

- 基础功能---**按title搜索笔记的实现**
1. 在listview中添加SearchView
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    xmlns:android="http://schemas.android.com/apk/res/android">
    <android.support.v7.widget.SearchView
        android:id="@+id/sv"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        >

    </android.support.v7.widget.SearchView>

    <ListView
        android:id="@+id/tv"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</LinearLayout>
```
2. 在NoteList.java中添加searchview实现函数
```
 private void SearchView(){
        searchView=findViewById(R.id.sv);
        searchView.onActionViewExpanded();
        searchView.setQueryHint("搜索笔记");
        searchView.setSubmitButtonEnabled(true);
        searchView.setOnQueryTextListener(new SearchView.OnQueryTextListener() {
            @Override
            public boolean onQueryTextSubmit(String s) {
                return false;
            }

            @Override
            public boolean onQueryTextChange(String s) {
                if(!s.equals("")){
                    String selection=NotePad.Notes.COLUMN_NAME_TITLE+" GLOB '*"+s+"*'";
                    updatecursor = getContentResolver().query(
                            getIntent().getData(),            // Use the default content URI for the provider.
                            PROJECTION,                       // Return the note ID and title for each note.
                            selection,                             // No where clause, return all records.
                            null,                             // No where clause, therefore no where column values.
                            NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                    );
                    if(updatecursor.moveToNext())
                        Log.i("daawdwad",selection);
                }
               else {
                    updatecursor = getContentResolver().query(
                            getIntent().getData(),            // Use the default content URI for the provider.
                            PROJECTION,                       // Return the note ID and title for each note.
                            null,                             // No where clause, return all records.
                            null,                             // No where clause, therefore no where column values.
                            NotePad.Notes.DEFAULT_SORT_ORDER  // Use the default sort order.
                    );
                }
                adapter.swapCursor(updatecursor);

               // adapter.notifyDataSetChanged();
                return false;
            }
        });
    }
```
#### 显示效果
![搜索实现](https://img-blog.csdnimg.cn/20190509183555465.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTMxNTI5NA==,size_16,color_FFFFFF,t_70 "基于title的搜索实现")

- 附加功能---**UI美化**和**更换背景颜色**
1. 在AndroidManifest中找到NoteList的activity标签中更换一个Theme
```
android:theme="@style/AlertDialog.AppCompat.Light"
```
2. 先在契约类NotePad里定义颜色常量，再在系统中预定于好五种颜色，根据颜色对应不int值选择要显示的颜色，契约类中的定义：
```
 public static final String COLUMN_NAME_BACK_COLOR = "color";
        public static final int DEFAULT_COLOR = 0; //白
        public static final int YELLOW_COLOR = 1; //黄
        public static final int BLUE_COLOR = 2; //蓝
        public static final int GREEN_COLOR = 3; //绿
        public static final int RED_COLOR = 4; //红
        
   ```
   3. 然后在数据库中添加相应的颜色字段，以便将背景颜色保存在数据库中进行存储
   ```
     @Override
        public void onCreate(SQLiteDatabase db) {
            db.execSQL("CREATE TABLE " + NotePad.Notes.TABLE_NAME + " ("
                    + NotePad.Notes._ID + " INTEGER PRIMARY KEY,"
                    + NotePad.Notes.COLUMN_NAME_TITLE + " TEXT,"
                    + NotePad.Notes.COLUMN_NAME_NOTE + " TEXT,"
                    + NotePad.Notes.COLUMN_NAME_CREATE_DATE + " INTEGER,"
                    + NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE + " Text,"
                    + NotePad.Notes.COLUMN_NAME_BACK_COLOR + " INTEGER"//color
                    + ");");
        }
  ```
 	 这里要特别注意每更改一次数据库中表时要换数据库的版本号
```
 /**
     * The database version
     */
    private static final int DATABASE_VERSION = 3;
 ```
 4. 接下来在NotePadProvider中的static模块添加对颜色的处理，也就是添加将颜色存进数据库的操作
 ```
    //color
        sNotesProjectionMap.put(
                NotePad.Notes.COLUMN_NAME_BACK_COLOR,
                NotePad.Notes.COLUMN_NAME_BACK_COLOR);
```
insert()函数里添加
```
  // 新建笔记时背景默认为白色
        if (values.containsKey(NotePad.Notes.COLUMN_NAME_BACK_COLOR) == false) {
            values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, NotePad.Notes.DEFAULT_COLOR);
        }
```
3. 再自己定义一个CursorAdapter继承SimpleCursorAdapter，既能完成cursor读取的数据库内容填充到item，又能将颜色填充：
```
package com.example.mynotepad;

import android.content.Context;
import android.database.Cursor;
import android.graphics.Color;
import android.view.View;
import android.widget.SimpleCursorAdapter;

public class MyCursorAdapter extends SimpleCursorAdapter {
    public MyCursorAdapter(Context context, int layout, Cursor c,
                           String[] from, int[] to) {
        super(context, layout, c, from, to);
    }
    @Override
    public void bindView(View view, Context context, Cursor cursor){
        super.bindView(view, context, cursor);
        //从数据库中读取的cursor中获取笔记列表对应的颜色数据，并设置笔记颜色
        int x = cursor.getInt(cursor.getColumnIndex(NotePad.Notes.COLUMN_NAME_BACK_COLOR));
        /**
         * 白 255 255 255
         * 黄 247 216 133
         * 蓝 165 202 237
         * 绿 161 214 174
         * 红 244 149 133
         */
        switch (x){
            case NotePad.Notes.DEFAULT_COLOR:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
            case NotePad.Notes.YELLOW_COLOR:
                view.setBackgroundColor(Color.rgb(247, 216, 133));
                break;
            case NotePad.Notes.BLUE_COLOR:
                view.setBackgroundColor(Color.rgb(165, 202, 237));
                break;
            case NotePad.Notes.GREEN_COLOR:
                view.setBackgroundColor(Color.rgb(161, 214, 174));
                break;
            case NotePad.Notes.RED_COLOR:
                view.setBackgroundColor(Color.rgb(244, 149, 133));
                break;
            default:
                view.setBackgroundColor(Color.rgb(255, 255, 255));
                break;
        }
    }
}
```
6. 添加一个note_color.xml布局文件实现更改背景颜色和NoteColor处理更换背景颜色操作
```
package com.example.mynotepad;

import android.app.Activity;
import android.content.ContentValues;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.view.View;

public class NoteColor extends Activity {
    private Cursor mCursor;
    private Uri mUri;
    private int color;
    private static final int COLUMN_INDEX_TITLE = 1;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.note_color);
        //从NoteEditor传入的uri
        mUri = getIntent().getData();
        mCursor = managedQuery(
                mUri,        // The URI for the note that is to be retrieved.
                PROJECTION,  // The columns to retrieve
                null,        // No selection criteria are used, so no where columns are needed.
                null,        // No where columns are used, so no where values are needed.
                null         // No sort order is needed.
        );
    }
    @Override
    protected void onResume(){
        //执行顺序在onCreate之后
        if (mCursor != null) {
            mCursor.moveToFirst();
            color = mCursor.getInt(COLUMN_INDEX_TITLE);
        }
        super.onResume();
    }
    @Override
    protected void onPause() {
        //执行顺序在finish()之后，将选择的颜色存入数据库
        super.onPause();
        ContentValues values = new ContentValues();
        values.put(NotePad.Notes.COLUMN_NAME_BACK_COLOR, color);
        getContentResolver().update(mUri, values, null, null);
    }
    public void white(View view){
        color = NotePad.Notes.DEFAULT_COLOR;
        finish();
    }
    public void yellow(View view){
        color = NotePad.Notes.YELLOW_COLOR;
        finish();
    }
    public void blue(View view){
        color = NotePad.Notes.BLUE_COLOR;
        finish();
    }
    public void green(View view){
        color = NotePad.Notes.GREEN_COLOR;
        finish();
    }
    public void red(View view){
        color = NotePad.Notes.RED_COLOR;
        finish();
    }

}

```
```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="horizontal" android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageButton
        android:id="@+id/color_white"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorWhite"
        android:onClick="white"/>
    <ImageButton
        android:id="@+id/color_yellow"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorYellow"
        android:onClick="yellow"/>
    <ImageButton
        android:id="@+id/color_blue"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorBlue"
        android:onClick="blue"/>
    <ImageButton
        android:id="@+id/color_green"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorGreen"
        android:onClick="green"/>
    <ImageButton
        android:id="@+id/color_red"
        android:layout_width="0dp"
        android:layout_height="50dp"
        android:layout_weight="1"
        android:background="@color/colorRed"
        android:onClick="red"/>
</LinearLayout>
```
7.在NoteList中的projection中添加颜色字段
```
/**
     * The columns needed by the cursor adapter
     */
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,
            //扩展  颜色
            NotePad.Notes.COLUMN_NAME_BACK_COLOR,
    };
 ```
 并且将NoteList中用的SimpleCursorAdapter改使用MyCursorAdapter：
 ```
 adapter = new MyCursorAdapter(
        this,
        R.layout.noteslist_item,
        cursor,
        dataColumns,
        viewIDs
    );
 ```
 8. 在editor_options_menu中添加更换背景颜色的按钮,和按钮的响应
 ```
  <item android:id="@+id/menu_color"
        android:title="@string/menu_color"
        android:icon="@drawable/change_background_color"
        app:showAsAction="always" >

    </item>
  ```
  在NoteEditor中找到onOptionsItemSelected()方法，在菜单的switch中添加：
  ```
  //换背景颜色选项
            case R.id.menu_color:
                changeColor();
                break;
  ```
  在NoteEditor中添加changecolor()
  ```
  private final void changeColor() {
        Intent intent = new Intent(null,mUri);
        intent.setClass(NoteEditor.this,NoteColor.class);
        NoteEditor.this.startActivity(intent);
    }
```
#### 显示效果
![更换背景](https://img-blog.csdnimg.cn/20190509190706706.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTMxNTI5NA==,size_16,color_FFFFFF,t_70 "更换背景")

- 附加功能--- **更改字体大小和颜色**
1. 在editor_options_menu中添加相应的按钮
```
 <item
        android:id="@+id/font_size"
        android:title="@string/font_size">
  <menu>
            <group>
                <item
                    android:id="@+id/font_10"
                    android:title="@string/font10"
                    />

                <item
                    android:id="@+id/font_16"
                    android:title="@string/font16" />
                <item
                    android:id="@+id/font_20"
                    android:title="@string/font20" />
            </group>
        </menu>
    </item>

    <item
        android:title="@string/font_color"
        android:id="@+id/font_color"
        >
        <menu>
            <group>
                <item
                    android:id="@+id/red_font"
                    android:title="@string/red_title" />
                <item
                    android:title="@string/black_title"
                    android:id="@+id/black_font"/>
            </group>
        </menu>
    </item>

 ```
 在NoteEditor中的onOptionsItemSelected(MenuItem item) 中添加
 ```
  case R.id.font_10:

                mText.setTextSize(20);
                Toast toast =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast.show();
              //  finish();
                break;

            case R.id.font_16:
                mText.setTextSize(32);
                Toast toast2 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast2.show();
              //  finish();
                break;
            case R.id.font_20:
                mText.setTextSize(40);
                Toast toast3 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast3.show();
                //finish();
                break;
            case R.id.red_font:
                mText.setTextColor(Color.RED);
                Toast toast4 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast4.show();
              //  finish();
                break;
            case R.id.black_font:
                mText.setTextColor(Color.BLACK);
                Toast toast5 =Toast.makeText(NoteEditor.this,"修改成功",Toast.LENGTH_SHORT);
                toast5.show();
               // finish();
                break;
   
```
#### 显示效果
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190509191320667.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80MTMxNTI5NA==,size_16,color_FFFFFF,t_70)
