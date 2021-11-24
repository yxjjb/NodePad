# NotePad
# 实验内容
# 一.基本功能
## 1.时间戳
![图片](https://github.com/yxjjb/ThirdTestInterfaceComponent/blob//main/picture/时间戳.png)

## 代码分析
### （1）首先在布局上先加一个TextView来承载时间戳
### （2）在NotesList中的PROJECTION 里添加NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//添加修改时间
### （3）在NoteList的 dataColumns 和 viewIDs 添加上时间戳和承载时间戳的TextView的id
        // The names of the cursor columns to display in the view, initialized to the title column
        String[] dataColumns = { NotePad.Notes.COLUMN_NAME_TITLE, NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE} ;
        // The view IDs that will display the cursor columns, initialized to the TextView in
        // noteslist_item.xml
        int[] viewIDs = { android.R.id.text1, R.id.text2};
### （4）修改一下时间的显示格式 
        long now = System.currentTimeMillis();
        Date date = new Date(now);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateFormat = simpleDateFormat.format(date);
        
## 2.搜索功能
![图片](https://github.com/yxjjb/ThirdTestInterfaceComponent/blob//main/picture/搜索.png)

## 代码分析
### （1）布局显示标题和时间外加一个图片
### （2）
    public class NoteSearch extends Activity implements SearchView.OnQueryTextListener{
    ListView listview;
    SQLiteDatabase sqLiteDatabase;
    private static final String[] PROJECTION = new String[] {
            NotePad.Notes._ID, // 0
            NotePad.Notes.COLUMN_NAME_TITLE, // 1
            NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE, // 修改时间
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
        int[] viewIDs = { R.id.text3, R.id.text4};
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

