## SimpleDateFormat为何线程不安全

```
用到了成员变量calendar，在并发情况下，对成员变量的操作是线程不安全的。
```

## 如何解决SimpleDateFormat的不安全问题

- 使用java8的新API--DateTimeFormatter，线程安全的api

  ```
          //字符串转日期
          String dateString = "2018年06月20日";
          DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy年MM月dd日");
          LocalDate date = LocalDate.parse(dateString, formatter);
          System.out.println(date);
  
  
          //日期转字符串
          LocalDateTime now = LocalDateTime.now();
          String nowStr = now.format(formatter);
          System.out.println(nowStr);
  ```

- 使用ThreadLocal—线程独享比方法独享减少了创建对象的开销

  ```
  private static ThreadLocal<DateFormat> threadLocal = new ThreadLocal<DateFormat>() {
          @Override
          protected DateFormat initialValue() {
              return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
          }
      };
  
      public static Date parse(String dateStr) throws ParseException {
          return threadLocal.get().parse(dateStr);
      }
  
      public static String format(Date date) {
          return threadLocal.get().format(date);
      }
  ```

- 使用同步—并发度差

  ```
  private static SimpleDateFormat sdf = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
  
      public static String formatDate(Date date) throws ParseException {
          synchronized(sdf) {
              return sdf.format(date);
          }  
      }
  
      public static Date parse(String strDate) throws ParseException {
          synchronized(sdf) {
              return sdf.parse(strDate);
          }
      }
  ```
