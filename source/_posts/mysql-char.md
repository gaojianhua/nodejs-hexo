title: mysql去除字段中的换行和回车符
date: 2017-04-09 19:18:10
tags:
- 乡村销客
- 大数据
categories: mysql
---

## 解决方法
UPDATE tablename SET  field = REPLACE(REPLACE(field, CHAR(10), ”), CHAR(13), ”);

<!-- more -->

char(10):  换行符
char(13):  回车符

问题产生原因：
      2种方法生成excel模式的报表：
      1）手动生成
           将表中的数据导出，生成CSV文件。
          用mysqldump 导出数据  
              #mysqldump -u xxx -p --tab=/tmp/ --fields-terminated-by="#" DBName TBName
              将会在tmp目录下生成TBName.txt 文件。
          在EXCEL中导入生成的txt文件
     2）直接生成csv格式文件
           mysqldump -u samu -p -T --fields-terminated-by=","  --fields-enclosed-by="" 
           --lines-terminated-by="\n"  --fields-escaped-by=""  test Customer
          或者：
          mysqldump -u samu -p --tab=/tmp/ --fields-terminated-by=","  --fields-enclosed-by="" 
          --lines-terminated-by="\n"  --fields-escaped-by=""  test Customer
          但是，无论上面哪一种方法，如果表的某个列里包含回车符或者换行符，
          那么生成的CSV文件或者进行excel导入，都会将原本的1行数据，拆分成2行。
          因为CSV或者excel导入，是按数据的行来认定数据条数。
         所以，必须在此之前，将字段中的回车符或者换行符，进行替换。

         