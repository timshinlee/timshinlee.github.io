1. SharedPreference
2. Internal Storage
3. External Storage
4. Sqlite
5. Network

# SharedPreference
使用场景：保存少量， 关键性的数据
数据格式：xml key:values
保存路径：data/data/package_name/shared_prefs/your_preference_name.xml
```
        final SharedPreferences myPrefs = getSharedPreferences("cumstom_prefs", MODE_PRIVATE);
        final SharedPreferences.Editor editor = myPrefs.edit();
        editor.putInt("age", 15);
        editor.putString("name","小明");
        editor.apply();

	final String name = myPrefs.getString("name", "匿名");
```
# Internal Storage
## 写入数据
保存路径：data/data/package_name/files/file_name.file_extension
打开输出流：openFileOutput
```
        final String editContent = editText.getText().toString();
        FileOutputStream fos = null;
        try {
            fos = openFileOutput("my_file.txt", MODE_PRIVATE);
            fos.write(editContent.getBytes(Charset.forName("utf-8")));
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                if (fos != null) {
                    fos.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
```
## 读取数据
打开输入流：openFileInput
```
        BufferedReader br = null;
        try {
            FileInputStream fis = openFileInput("my_file.txt");
            br = new BufferedReader(new InputStreamReader(fis));
            String line = null;
            StringBuilder result = new StringBuilder();
            while (null != (line = br.readLine())) {
                result.append(line);
            }
        } catch (FileNotFoundException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(br!=null){
                try {
                    br.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
```
# External Storage
# Sqlite
Sqlite特点：轻量，每个应用都有独立数据库。跨平台	支持大部分的Sql语句
保存路径：data/data/package_name/dataBase/database_name.db
# Network
