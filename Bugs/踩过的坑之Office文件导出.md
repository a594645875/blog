#### Word导出时特殊字符问题
使用freemarker和word的xml,导出带图片的word
1.特殊字符需进行转义"& < > ' \",有工具类
2.保存xml模板时,尽量选择低版本的xml,保证兼容性
3.向xml2003中插入Base64码的位置不能换行.
例子:<xx>Base64</xx>
