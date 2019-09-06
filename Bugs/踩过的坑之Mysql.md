#### mysql的时区错误问题
问题：`The server time zone value 'ÖÐ¹ú±ê×¼Ê±¼ä' is unrecognized or represents more than one time zone....`
解决方案:在数据库中执行以下操作
`show variables like '%time_zone%';
set global time_zone='+8:00';`
