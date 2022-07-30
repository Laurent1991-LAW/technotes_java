# 字符串转化为时区概念时间

```java
public static void main(String[] args) throws ParseException {
    String str = "2022-05-05 12:58:23";
    SimpleDateFormat sdf =  new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
    sdf.setTimeZone(TimeZone.getTimeZone("GMT+8"));  // 时区设置可省略，默认为系统时区
    Date date = sdf.parse(str);
    System.out.println(date);
}

//打印结果如下，CST为美国、澳大利亚、古巴或中国的标准时间
//Thu May 05 00:58:23 CST 2022
```

<br/>

# 应用：计算更新时间

**场景介绍：** 我们在实际业务中，经常需要获取发布、更新时间，比如微博中的评论会显示：25秒前 /  4分钟前 / 5小时前 / 3天前 [25 second(s) ago / 4 minute(s) ago / 5 hour(s) ago / 3 day(s) ago ]，该国际化同样在后台实现。

```java
public class TimeGapTest {
	
    public static void main(String[] args) throws ParseException {
        // 国际化标签 一般从前端传回的用户上下文环境获取
        boolean enFlag = false;
        // LocalDateTime类parse方法需要标准字符串格式时间, 即日期与时间之间带有一个字母T
        String updateDate = "2018-12-30T19:34:50.63";
        LocalDateTime dateTime = LocalDateTime.parse(updateDate);
        // Duration.between()所需参数为LocalDateTime类型而不是Date
        Duration duration = Duration.between(dateTime, LocalDateTime.now());
        long seconds = duration.getSeconds();
        String res = getTimeGat(seconds, enFlag);
        System.out.println(res);
    }

    private static String getTimeGat(long seconds, boolean enFlag) {
        if (enFlag) {
            if(seconds<60) {
                return seconds + " " + TimeUnitsPattern.SECOND.EN;
            }
            if(seconds<60*60) {
                return seconds/60 +" "+TimeUnitsPattern.MINUTE.EN;
            }
            if(seconds<60*60*24) {
                return seconds/(60*60) +" "+TimeUnitsPattern.HOUR.EN;
            }
            if(seconds<60*60*24*30) {
                return seconds/(60*60*24) +" "+TimeUnitsPattern.DAY.EN;
            }
            if(seconds<60*60*24*30*12) {
                return seconds/(60*60*24*30) +" "+TimeUnitsPattern.MONTH.EN;
            }
            if(seconds>60*60*24*30*12) {
                return seconds/(60*60*24*30*12) +" "+TimeUnitsPattern.YEAR.EN;
            }
        } else {
            if(seconds<60) {
                return seconds + TimeUnitsPattern.SECOND.CH;
            }
            if(seconds<60*60) {
                return seconds/60 + TimeUnitsPattern.MINUTE.CH;
            }
            if(seconds<60*60*24) {
                return seconds/(60*60) + TimeUnitsPattern.HOUR.CH;
            }
            if(seconds<60*60*24*30) {
                return seconds/(60*60*24) + TimeUnitsPattern.DAY.CH;
            }
            if(seconds<60*60*24*30*12) {
                return seconds/(60*60*24*30) + TimeUnitsPattern.MONTH.CH;
            }
            if(seconds>60*60*24*30*12) {
                return seconds/(60*60*24*30*12) + TimeUnitsPattern.YEAR.CH;
            }
        }
        return "No result!";
    }
    
    // 时间单位枚举类
    enum TimeUnitsPattern {

        SECOND ("秒前","second(s) ago"),
        MINUTE ("分前","minute(s) ago"),
        HOUR ("小时前","hour(s) ago"),
        DAY ("天前","day(s) ago"),
        MONTH ("月前","month(s) ago"),
        YEAR ("年前","year(s) ago");

        private String CH;
        private String EN;

        TimeUnitsPattern(String ch, String en) {
            this.CH = ch;
            this.EN = en;
        }
    }
}
```



<br/>

# 应用：用Calendar类计算间隔月份

背景：若依据上例的计算方法获取月份差，每月天数的差异将会被忽略，此时可以利用Calendar API对两个日期的间隔月份进行精确获取



<br/>

# 应用：生成附件编码

**背景介绍：**获取格式如NEG202205150005、SRM202205150003、CON202205150045的附件编码，前三位依据附件类型而定，中间为上传日期，最后四位为当日同类附件自增个数。

```JAVA
public class DocRefTest {
    public synchronized String getDocRef(int code) {  // 参数为附件类型
        StringBuilder builder = new StringBuilder();
        // 获取类型
        String abr = FileTypeEnum.getABREVIATION(code);
        // 获取日期-方法一     
        String date = new SimpleDateFormat("yyyyMMdd").format(new Date());
        /* 获取日期-方法二
        DateTimeFormatter dtf = DateTimeFormatter.ofPattern("yyyyMMdd");
        String date = dtf.format(LocalDateTime.now()); 
        */
        // 类型+日期前缀
        String prefixe = builder.append(abr)
                .append(date).toString();
        // 获取当前前缀的附件编码集合
        List<String> refs = fileMapper.findDocRefByPrefixe(prefixe+"%");
        if (refs == null || refs.isEmpty()) {
            return prefixe + "0001";
        } else {
            // 获取编码后四位中的最大值
            Integer max = refs.stream().map(ref -> {
                String suffix = ref.substring(ref.length() - 4);
                return Integer.parseInt(suffix);
            }).collect(Collectors.maxBy(Integer::compare)).get();
            max++;
            String res = max.toString();
            while (res.length() < 4) {
                res = "0" + res;
            }
            return prefixe + res;
        }
    }
    
    enum FileTypeEnum {

        /* 合同 */
        CONTRAT(1, "CON"),
        /* SRM */
        SRM(2, "SRM"),
        /* 会谈记录 */
        NEGOCIATION_RECORD(3, "NEG");

        private final int TYPE_CODE;
        private final String ABREVIATION;

        FileTypeEnum(int code, String abre) {
            this.TYPE_CODE = code;
            this.ABREVIATION = abre;
        }

        // 验证附件编码是否合法
        private boolean isCodeValid(int code) {
            for (FileTypeEnum each : FileTypeEnum.values()) {
                if (each.TYPE_CODE == code) return true;
            }
            return false;
        }

        public String getABREVIATION(int code) {
            for (FileTypeEnum each : FileTypeEnum.values()) {
                if (each.TYPE_CODE == code) return each.ABREVIATION;
            }
            return null;
        }
    }
}
```

