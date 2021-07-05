

文档

- 页眉
  - 段落/表格

- 段落
  - Runs
    - Run(文档的最小单元)
- 表格
  - 行
    - 列（格，每一个的内容相当于一个完成的文档）



获取所有段落：List<XWPFParagraph> paragraphs = word.getParagraphs();

获取一个段落中的所有Runs：List<XWPFRun> xwpfRuns = xwpfParagraph.getRuns();

获取一个Runs中的一个Run：XWPFRun run = xwpfRuns.get(index);



获取所有表格：List<XWPFTable> xwpfTables = doc.getTables();

获取一个表格中的所有行：List<XWPFTableRow> xwpfTableRows = xwpfTable.getRows();

获取一行中的所有列：List<XWPFTableCell> xwpfTableCells = xwpfTableRow.getTableCells();

获取一格里的内容：List<XWPFParagraph> paragraphs = xwpfTableCell.getParagraphs();