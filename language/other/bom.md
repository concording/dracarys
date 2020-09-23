## csv文件导入数据首行首列有BOM字符


`str.trim().replaceAll("\uFEFF", ""));`