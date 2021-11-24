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

