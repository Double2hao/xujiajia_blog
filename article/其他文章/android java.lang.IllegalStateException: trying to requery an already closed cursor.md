#android java.lang.IllegalStateException: trying to requery an already closed cursor
**错误提示：**

 

Java.lang.RuntimeException: Unable to resume activity {com.lenovo.leos.memowidget/com.lenovo.leos.notepad.NoteEditor}: java.lang.IllegalStateException: trying to requery an already closed cursor

 

 

**可能错误的使用方法：**

query(android.net.Uri, String[], String, String[], String) startManagingCursor(Cursor)

 

由activity在通过query获取了Cursor之后用startManagingCursor来管理Cursor的生命周期的，那么每一次调用完毕之后Cursor也会相应的被关闭；由此从history menu tab进入的时候则可能因为Cursor被关闭了而导致异常。

 

 

 

**解决办法：**

不使用startManagingCursor(Cursor)来管理Cursor的生命周期，自己使用.close()管理。