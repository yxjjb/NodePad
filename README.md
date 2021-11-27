# NotePad
# 实验内容
# 一.基本功能
## 1.时间戳
![images](https://github.com/yxjjb/NodePad/blob/main/picture/1.png)

## 代码分析
### （1）首先在布局上先加一个TextView来承载时间戳
### （2）在NotesList中的PROJECTION 里添加NotePad.Notes.COLUMN_NAME_MODIFICATION_DATE,//添加修改时间
### （3）在NoteList的 dataColumns 和 viewIDs 添加上时间戳和承载时间戳的TextView的id
![images](https://github.com/yxjjb/NodePad/blob/main/picture/5.png)
        
### （4）修改一下时间的显示格式 
        long now = System.currentTimeMillis();
        Date date = new Date(now);
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
        String dateFormat = simpleDateFormat.format(date);
        
## 2.搜索功能
![图片](https://github.com/yxjjb/NodePad/blob/main/picture/2.png)

## 代码分析
### （1）布局显示标题和时间外加一个图片
### （2）搜索实现
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

### （3）最后还需要在Mainfest 中添加 
        <activity android:name=".NoteSearch"></activity>

# 二.扩展功能
## 1.背景轻音乐播放器
![图片](https://github.com/yxjjb/NodePad/blob/main/picture/3.png)
![图片](https://github.com/yxjjb/NodePad/blob/main/picture/4.png)

## 代码分析
### 
    public IBinder onBind(Intent intent) {
        return null;
    }
    //在此方法中服务被创建
    @Override
    public void onCreate() {
        super.onCreate();
        if (mediaPlayer==null){
            mediaPlayer=new MediaPlayer();

            //为播放器添加播放完成时的监听器
            mediaPlayer.setOnCompletionListener(new MediaPlayer.OnCompletionListener() {
                @Override
                public void onCompletion(MediaPlayer mp) {
                    //发送广播到MainActivity
                    Intent intent=new Intent();
                    intent.setAction("com.complete");
                    sendBroadcast(intent);
                }
            });
        }
    } 
    
    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        switch (intent.getIntExtra("type",-1)){
            case NotesList.PLAT_MUSIC:
                if (isStop){
                    //重置mediaplayer
                    mediaPlayer.reset();
                    //将需要播放的资源与之绑定
                    mediaPlayer=MediaPlayer.create(this,R.raw.music);
                    //开始播放
                    mediaPlayer.start();
                    //是否循环播放
                    mediaPlayer.setLooping(false);
                    isStop=false;
                }else if (!isStop&&mediaPlayer.isPlaying()&&mediaPlayer!=null){
                    mediaPlayer.start();
                }
                break;
            case NotesList.PAUSE_MUSIC:
                //播放器不为空，并且正在播放
                if (mediaPlayer!=null&&mediaPlayer.isPlaying()){
                    mediaPlayer.pause();
                }
                break;
            case NotesList.STOP_MUSIC:
                if (mediaPlayer!=null){
                    //停止之后要开始播放音乐
                    mediaPlayer.stop();
                    isStop=true;
                }
                break;
        }
        return START_NOT_STICKY;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }
}
### 添加一个 item 按钮
        //开始音乐
        case R.id.btn_startmusic:
            playingmusic(PLAT_MUSIC);
            Toast.makeText(this,"音乐开始播放", Toast.LENGTH_SHORT).show();
            break;
            
        private void playingmusic(int type) {
        //启动服务，播放音乐
        Intent intent=new Intent(this,PlayingMusicServices.class);
        intent.putExtra("type",type);
        startService(intent);
    }

### 最后还需要在Mainfest 中添加 
        <service android:name=".PlayingMusicServices"
            android:exported="true"
            android:enabled="true"/>
            
## 2.主界面和搜索界面的部分美化
![images](https://github.com/yxjjb/NodePad/blob/main/picture/1.png)
![图片](https://github.com/yxjjb/NodePad/blob/main/picture/2.png)

#### 3.更改每条记事条目的背景颜色（未实现）



