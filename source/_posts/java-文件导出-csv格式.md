---
title: java 文件导出.csv格式
date: 2019-08-28 15:54:52
tags:
---
使用：
```java
import com.diditech.iov.dataexporter.output.csv.CsvExporter;
import com.diditech.iov.exporter.utils.ColumnHeadbuilder;
public class csv{
    
 public void getStopPointsInfoCsv() {
     //下载文件位置
    String tempPath = localSavePath + userId + File.separator;
    //下载文件名称
    String downloadName = "停留点导出";
    //下载文件内容
    List<StopReportNew> resultList = new ArrayList<>();
    //开始csv导出
        //初始化导出容器
    CsvExporter exporter = ExporterUtils.initCsvExporter(downloadName, FILE_TYPE_CSV, tempPath);
        //csv列名
    exporter = ColumnHeadbuilder.makeHeader(exporter, ExporterEnum.StopReportNewExtend);
        //文件追加内容
    exporter.addBeanRows(resultList);
        //关闭导出
    exporter.finishExporting(true);
    }
    }
```

```java
package com.diditech.iov.core.utils.exporter;

import com.diditech.iov.core.exception.BusinessException;
import com.diditech.iov.dataexporter.output.csv.CsvExporter;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.core.io.DefaultResourceLoader;
import org.springframework.core.io.Resource;
import org.springframework.core.io.ResourceLoader;
import org.springframework.util.ResourceUtils;

import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.OutputStreamWriter;
import java.nio.channels.FileChannel;


public class ExporterUtils {

    private final static byte[] UTF8BOM = {(byte) 0xef, (byte) 0xbb, (byte) 0xbf};

    private final static Logger logger = LoggerFactory.getLogger(ExporterUtils.class);

    private final static ResourceLoader resourceLoader = new DefaultResourceLoader();


    public static CsvExporter initCsvExporter(String fileName, String fileType, String path){
        try {
            File tempPath = new File(path);
            if (!tempPath.exists()) {
                tempPath.mkdirs();
            }
            File targetFile = new File(path + File.separator + fileName + fileType);
            createFile(targetFile);
            return new CsvExporter(new OutputStreamWriter(new FileOutputStream(targetFile, true), "UTF-8"));
        } catch (IOException e) {
            logger.error("初始化导出器失败", e);
        }
        throw new BusinessException("初始化导出器失败");
    }


    public static void createFile(File targetFile) throws IOException{
        FileOutputStream fileOutputStream = null;
        try {
            fileOutputStream = new FileOutputStream(targetFile);
            fileOutputStream.write(UTF8BOM, 0, UTF8BOM.length);
            fileOutputStream.flush();
        } catch (Exception e) {
            logger.error("创建导出文件异常", e);
            throw new BusinessException("创建导出文件异常"+e.getMessage());
        }finally {
            if(fileOutputStream!=null){
                fileOutputStream.close();
            }
        }
    }


    public static InputStream findFileInputStream(String templateName){
        Resource resource = resourceLoader.getResource(ResourceUtils.CLASSPATH_URL_PREFIX + templateName);
        try {
            return resource.getInputStream();
        } catch (IOException e) {
            logger.error("初始化模板异常", e);
            throw new BusinessException("初始化模板异常");
        }
    }
    public static File findFileResouce(String templateName){
        try {
            return ResourceUtils.getFile(templateName);
        } catch (IOException e) {
            logger.error("初始化导出模板异常", e);
        }
        return null;
    }


    private static void copyFile(File source, File target) throws IOException {
        FileChannel inputChannel = null;
        FileChannel outputChannel = null;
        try {
            inputChannel = new FileInputStream(source).getChannel();
            outputChannel = new FileOutputStream(target).getChannel();
            outputChannel.transferFrom(inputChannel, 0, inputChannel.size());
        } finally {
            inputChannel.close();
            outputChannel.close();
        }
    }

}

```

```java

import com.diditech.iov.dataexporter.DataExporter;
import java.io.OutputStream;
import java.io.Writer;

public class CsvExporter extends DataExporter {
    public CsvExporter(OutputStream out) {
        this(new CsvExportOptions(), out);
    }

    public CsvExporter(CsvExportOptions options, OutputStream out) {
        super(new CsvWriter(options, out));
    }

    public CsvExporter(Writer out) {
        this(new CsvExportOptions(), out);
    }

    public CsvExporter(CsvExportOptions csvExportOptions, Writer out) {
        super(new CsvWriter(csvExportOptions, out));
    }

    public CsvExporter() {
        this((OutputStream)System.out);
    }

    public CsvExporter(CsvExportOptions options) {
        this(options, (OutputStream)System.out);
    }

    public CsvExportOptions getCsvExportOptions() {
        return (CsvExportOptions)this.getDataWriter().getOptions();
    }

    public void finishExporting() {
        super.finishExporting();
    }
}
```

```java
import com.diditech.iov.dataexporter.model.BeanRow;
import com.diditech.iov.dataexporter.model.Column;
import com.diditech.iov.dataexporter.model.DataExporterCallback;
import com.diditech.iov.dataexporter.model.Row;
import com.diditech.iov.dataexporter.model.StringColumn;
import com.diditech.iov.dataexporter.model.Table;
import com.diditech.iov.dataexporter.util.Util;
import java.io.PrintWriter;
import java.util.Arrays;
import java.util.Iterator;
import java.util.List;

public abstract class DataExporter {
    private Table table = new Table();
    private DataWriter dataWriter = null;
    private boolean startedExporting = false;

    protected DataExporter(DataWriter dataWriter) {
        Util.checkForNotNull(dataWriter, "datawriter");
        this.dataWriter = dataWriter;
    }

    public DataExporterCallback getCallback() {
        return this.table.getCallback();
    }

    public void setCallback(DataExporterCallback callback) {
        this.table.setCallback(callback);
    }

    public void setOutputStream(PrintWriter out) {
        this.dataWriter.setOutputStream(out);
    }

    public DataWriter getDataWriter() {
        return this.dataWriter;
    }

    public ExportOptions getOptions() {
        return this.dataWriter.getOptions();
    }

    public DataExporter addColumn(String... columns) {
        Util.checkForNotNull(columns, "columns");
        String[] var2 = columns;
        int var3 = columns.length;

        for(int var4 = 0; var4 < var3; ++var4) {
            String column = var2[var4];
            this.table.addColumn(new StringColumn(column));
        }

        return this;
    }

    public DataExporter addColumns(Column... columns) {
        Util.checkForNotNull(columns, "columns");
        this.checkNotExporting();
        Column[] var2 = columns;
        int var3 = columns.length;

        for(int var4 = 0; var4 < var3; ++var4) {
            Column column = var2[var4];
            this.table.addColumn(column);
        }

        return this;
    }

    public DataExporter addRow(Object... rowValues) {
        Util.checkForNotNull(rowValues, "rowValues");
        Row row = new Row();
        row.add(rowValues);
        this.addRows(row);
        return this;
    }

    public DataExporter addRows(Row... rows) {
        Util.checkForNotNull(rows, "rows");
        this.startExporting();
        this.dataWriter.writeRows(this.table, Arrays.asList(rows));
        return this;
    }

    public DataExporter addBeanRows(Object... beans) {
        Util.checkForNotNull(beans, "beans");
        Object[] var2 = beans;
        int var3 = beans.length;

        for(int var4 = 0; var4 < var3; ++var4) {
            Object object = var2[var4];
            this.addRows(new BeanRow(object));
        }

        return this;
    }

    public DataExporter addBeanRows(List beans) {
        Util.checkForNotNull(beans, "beans");
        Iterator var2 = beans.iterator();

        while(var2.hasNext()) {
            Object object = var2.next();
            this.addRows(new BeanRow(object));
        }

        return this;
    }

    public synchronized void startExporting() {
        if (!this.startedExporting) {
            this.startedExporting = true;
            this.dataWriter.beforeTable(this.table);
            this.dataWriter.writeHeader(this.table);
        }
    }

    public void finishExporting() {
        this.finishExporting(false);
    }

    public void finishExporting(boolean endFlag) {
        if (!this.startedExporting && !endFlag) {
            throw new IllegalStateException("Data export has not been started to finish!");
        } else {
            this.dataWriter.afterTable(this.table);
            this.dataWriter.flush();
            this.dataWriter.finishExporting(endFlag);
            this.startedExporting = false;
        }
    }

    private void checkNotExporting() {
        if (this.startedExporting) {
            throw new IllegalStateException("DataExporter has already started exporting the data and hence new columns cannot be added.");
        }
    }
}
```

```java
import com.diditech.iov.dataexporter.DataExportException;
import org.apache.commons.beanutils.PropertyUtils;

public class BeanRow extends Row {
    private Object bean = null;

    public BeanRow(Object bean) {
        this.bean = bean;
    }

    public Object getCellValue(String columnName) {
        try {
            return PropertyUtils.getProperty(this.bean, columnName);
        } catch (Exception var3) {
            throw new DataExportException("Exception while reading the property " + columnName + " from bean " + this.bean + " of type " + this.bean.getClass().getName(), var3);
        }
    }

    public Object getBean() {
        return this.bean;
    }
}
```

```java
public class DataExportException extends RuntimeException {
    private static final long serialVersionUID = 1L;

    public DataExportException() {
    }

    public DataExportException(String message, Throwable cause) {
        super(message, cause);
    }

    public DataExportException(String message) {
        super(message);
    }

    public DataExportException(Throwable cause) {
        super(cause);
    }
}

```

```java
import com.diditech.iov.dataexporter.model.AlignType;
import com.diditech.iov.dataexporter.util.Util;
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class TextAligner {
    public TextAligner() {
    }

    public int getRowHeight(int width, String data, AlignType alignType) {
        List<String> alignedStrings = this.align(width, data.length(), alignType, data);
        int rowHeight = 0;
        Iterator var6 = alignedStrings.iterator();

        while(var6.hasNext()) {
            String line = (String)var6.next();
            if (line.trim().length() > 0) {
                ++rowHeight;
            }
        }

        return Math.max(1, rowHeight);
    }

    public List<String> align(int width, int height, AlignType align, String data) {
        return this.align(width, height, align, data, " ");
    }

    public List<String> align(int width, int height, AlignType align, String data, String filler) {
        Util.checkForNotNull(align, "align");
        if (height > 0 && width > 0) {
            if (data == null) {
                data = "";
            }

            int totalChars = width * height;
            if (data.length() > totalChars) {
                throw new IllegalArgumentException("Data given (" + data.length() + " chars) is bigger than cell size of " + totalChars + " chars (width " + width + " x height " + height + ")");
            } else {
                List<String> compacted = new ArrayList();
                int i;
                int numerOfFillers;
                if (data.length() <= width) {
                    int midwayIndex;
                    label226:
                    switch(align) {
                    case TOP_CENTER:
                    case TOP_LEFT:
                    case TOP_RIGHT:
                        compacted.add(data);
                        midwayIndex = 0;

                        while(true) {
                            if (midwayIndex >= height - 1) {
                                break label226;
                            }

                            compacted.add("");
                            ++midwayIndex;
                        }
                    case MIDDLE_CENTER:
                    case MIDDLE_LEFT:
                    case MIDDLE_RIGHT:
                        midwayIndex = (int)Math.ceil((double)((float)height / 2.0F)) - 1;
                        if (midwayIndex >= height) {
                            --midwayIndex;
                        }

                        for(i = 0; i < midwayIndex; ++i) {
                            compacted.add("");
                        }

                        compacted.add(data);
                        i = midwayIndex + 1;

                        while(true) {
                            if (i >= height) {
                                break label226;
                            }

                            compacted.add("");
                            ++i;
                        }
                    case BOTTOM_CENTER:
                    case BOTTOM_LEFT:
                    case BOTTOM_RIGHT:
                        for(i = 0; i < height - 1; ++i) {
                            compacted.add("");
                        }

                        compacted.add(data);
                    }
                } else {
                    String[] words = data.split("\\s");
                    List<String> wordsFinal = new ArrayList();
                    String[] var10 = words;
                    numerOfFillers = words.length;

                    int midwayIndex;
                    for(midwayIndex = 0; midwayIndex < numerOfFillers; ++midwayIndex) {
                        String word = var10[midwayIndex];
                        if (word.trim().length() >= width) {
                            wordsFinal.addAll(this.splitWord(word, width));
                        } else {
                            wordsFinal.add(word.trim());
                        }
                    }

                    StringBuilder sb = new StringBuilder();
                    int numerOfFillers = false;
                    int i;
                    label195:
                    switch(align) {
                    case TOP_CENTER:
                    case TOP_LEFT:
                    case TOP_RIGHT:
                        for(midwayIndex = 0; midwayIndex < wordsFinal.size(); ++midwayIndex) {
                            if ((sb.length() == 0 ? 0 : sb.length() + 1) + ((String)wordsFinal.get(midwayIndex)).length() <= width) {
                                if (sb.length() > 0) {
                                    sb.append(" ");
                                }

                                sb.append((String)wordsFinal.get(midwayIndex));
                            } else {
                                compacted.add(sb.toString());
                                sb = new StringBuilder();
                                sb.append((String)wordsFinal.get(midwayIndex));
                            }
                        }

                        if (sb.length() > 0) {
                            compacted.add(sb.toString());
                        }

                        numerOfFillers = height - compacted.size();
                        midwayIndex = 0;

                        while(true) {
                            if (midwayIndex >= numerOfFillers) {
                                break label195;
                            }

                            compacted.add("");
                            ++midwayIndex;
                        }
                    case MIDDLE_CENTER:
                    case MIDDLE_LEFT:
                    case MIDDLE_RIGHT:
                        midwayIndex = (int)Math.ceil((double)((float)wordsFinal.size() / 2.0F));
                        if (midwayIndex >= wordsFinal.size()) {
                            --midwayIndex;
                        }

                        for(i = midwayIndex; i >= 0; --i) {
                            if (sb.length() + ((String)wordsFinal.get(i)).length() <= width - 1) {
                                if (sb.length() > 0) {
                                    sb.insert(0, " ");
                                }

                                sb.insert(0, (String)wordsFinal.get(i));
                            } else {
                                compacted.add(0, sb.toString());
                                sb = new StringBuilder();
                                sb.insert(0, (String)wordsFinal.get(i));
                            }
                        }

                        if (sb.length() > 0) {
                            compacted.add(0, sb.toString());
                        }

                        sb = new StringBuilder();

                        for(i = midwayIndex + 1; i < wordsFinal.size(); ++i) {
                            if (sb.length() + ((String)wordsFinal.get(i)).length() <= width) {
                                if (sb.length() > 0) {
                                    sb.append(" ");
                                }

                                sb.append((String)wordsFinal.get(i));
                            } else {
                                compacted.add(sb.toString());
                                sb = new StringBuilder();
                                sb.append((String)wordsFinal.get(i));
                            }
                        }

                        if (sb.length() > 0) {
                            compacted.add(sb.toString());
                        }

                        numerOfFillers = Math.max(0, height - compacted.size()) / 2;

                        for(i = 0; i < numerOfFillers; ++i) {
                            compacted.add(0, "");
                        }

                        numerOfFillers = height - compacted.size();
                        i = 0;

                        while(true) {
                            if (i >= numerOfFillers) {
                                break label195;
                            }

                            compacted.add("");
                            ++i;
                        }
                    case BOTTOM_CENTER:
                    case BOTTOM_LEFT:
                    case BOTTOM_RIGHT:
                        for(i = wordsFinal.size() - 1; i >= 0; --i) {
                            if (sb.length() + ((String)wordsFinal.get(i)).length() <= width - 1) {
                                if (sb.length() > 0) {
                                    sb.insert(0, " ");
                                }

                                sb.insert(0, (String)wordsFinal.get(i));
                            } else {
                                compacted.add(0, sb.toString());
                                sb = new StringBuilder();
                                sb.insert(0, (String)wordsFinal.get(i));
                            }
                        }

                        if (sb.length() > 0) {
                            compacted.add(0, sb.toString());
                        }

                        numerOfFillers = height - compacted.size();

                        for(i = 0; i < numerOfFillers; ++i) {
                            compacted.add(0, "");
                        }
                    }
                }

                List<String> aligned = new ArrayList();

                for(i = 0; i < compacted.size(); ++i) {
                    int prefixCount = 0;
                    numerOfFillers = 0;
                    String tempData = (String)compacted.get(i);
                    if (tempData == null) {
                        tempData = "";
                    }

                    switch(align) {
                    case TOP_CENTER:
                    case MIDDLE_CENTER:
                    case BOTTOM_CENTER:
                        prefixCount = (width - tempData.length()) / 2;
                        numerOfFillers = width - prefixCount - tempData.length();
                        break;
                    case TOP_LEFT:
                    case MIDDLE_LEFT:
                    case BOTTOM_LEFT:
                        numerOfFillers = width - tempData.length();
                        break;
                    case TOP_RIGHT:
                    case MIDDLE_RIGHT:
                    case BOTTOM_RIGHT:
                        prefixCount = width - tempData.length();
                    }

                    aligned.add(Util.createSpacer(prefixCount) + tempData + Util.createSpacer(numerOfFillers));
                }

                return aligned;
            }
        } else {
            throw new IllegalArgumentException("Height or width cannot be less than or equal to zero.");
        }
    }

    private String substring(String data, int startIndex, int endIndex) {
        if (startIndex < 0) {
            startIndex = 0;
        }

        if (endIndex > data.length()) {
            endIndex = data.length();
        }

        return data.substring(startIndex, endIndex);
    }

    private List<String> splitWord(String word, int width) {
        List<String> words = new ArrayList();

        for(int startIndex = 0; startIndex < word.length(); startIndex += width) {
            words.add(this.substring(word, startIndex, startIndex + width).trim());
        }

        return words;
    }
}

```

```java
public enum AlignType {
    TOP_LEFT("top", "left"),
    TOP_CENTER("top", "center"),
    TOP_RIGHT("top", "right"),
    MIDDLE_LEFT("middle", "left"),
    MIDDLE_CENTER("middle", "center"),
    MIDDLE_RIGHT("middle", "right"),
    BOTTOM_LEFT("bottom", "left"),
    BOTTOM_CENTER("bottom", "center"),
    BOTTOM_RIGHT("bottom", "right");

    private String horAlign = null;
    private String verAlign = null;

    private AlignType(String verAlign, String horAlign) {
        this.verAlign = verAlign;
        this.horAlign = horAlign;
    }

    public String getHorizontalAlignment() {
        return this.horAlign;
    }

    public String getVerticalAlignment() {
        return this.verAlign;
    }

    public boolean isLeft() {
        return this.horAlign.equals("left");
    }

    public boolean isCenter() {
        return this.horAlign.equals("center");
    }

    public boolean isRight() {
        return this.horAlign.equals("right");
    }
}
```

```java
public class Util {
    public Util() {
    }

    public static String createSpacer(int length) {
        return createString(" ", length);
    }

    public static String createString(String chr, int length) {
        if (length <= 0) {
            return "";
        } else {
            StringBuilder sb = new StringBuilder();

            while(sb.length() < length) {
                sb.append(chr);
            }

            return sb.toString().substring(0, length);
        }
    }

    public static void checkForNotNull(Object object, String name) {
        if (object == null) {
            throw new IllegalArgumentException("The parameter '" + name + "' cannot be null");
        }
    }
}
```

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class Row {
    private List<Object> cells = new ArrayList();
    private List<Row> children = null;

    public Row() {
    }

    public Row(Object... rowValues) {
        this.add(rowValues);
    }

    public Row(List<? extends Object> rowValues) {
        if (rowValues == null) {
            throw new IllegalArgumentException("Parameter rowValues cannot be null");
        } else {
            Iterator var2 = rowValues.iterator();

            while(var2.hasNext()) {
                Object object = var2.next();
                this.add(object);
            }

        }
    }

    public Row add(Object... rowValues) {
        if (rowValues == null) {
            throw new IllegalArgumentException("Parameter rowValues cannot be null");
        } else {
            Object[] var2 = rowValues;
            int var3 = rowValues.length;

            for(int var4 = 0; var4 < var3; ++var4) {
                Object value = var2[var4];
                this.cells.add(value);
            }

            return this;
        }
    }

    public Object getCellValue(CellDetails cellDetails) {
        return this.cells.get(cellDetails.getColumnIndex());
    }

    public Object getCellValue(int index) {
        return this.cells.get(index);
    }

    public void setCellValue(int index, Object value) {
        this.cells.set(index, value);
    }

    public void addCellValue(Object value) {
        this.cells.add(value);
    }

    public void clearCellValues() {
        this.cells = new ArrayList();
    }

    public void setCellValues(List<Object> cells) {
        this.cells = cells;
    }

    public List<Object> getCellValues() {
        return this.cells;
    }

    public List<Row> getChildren() {
        return this.children;
    }

    public void setChildren(List<Row> children) {
        this.children = children;
    }

    public void addChild(Row child) {
        if (this.children == null) {
            this.children = new ArrayList();
        }

        this.children.add(child);
    }
}

```

```java
public class StringColumn extends Column {
    public StringColumn(String name) {
        super(name);
    }

    public StringColumn(String name, int width) {
        this(name, width, AlignType.MIDDLE_LEFT);
    }

    public StringColumn(String name, String title, int width) {
        this(name, title, width, AlignType.MIDDLE_LEFT);
    }

    public StringColumn(String name, int width, AlignType align) {
        super(name, width, align);
    }

    public StringColumn(String name, String title, int width, AlignType align) {
        super(name, title, width, align);
    }

    public int getMaxRowHeight(CellDetails cellDetails) {
        return cellDetails.getCellValue() == null ? 1 : this.getMaxRowHeight(cellDetails.getCellValue().toString());
    }

    public String format(CellDetails cellDetails) {
        return cellDetails.getCellValue() == null ? "" : cellDetails.getCellValue().toString();
    }
}
```

```java
import java.util.ArrayList;
import java.util.List;

public class Table {
    private DataExporterCallback callback = null;
    private List<Column> columns = new ArrayList();

    public Table() {
    }

    public DataExporterCallback getCallback() {
        return this.callback;
    }

    public void setCallback(DataExporterCallback callback) {
        this.callback = callback;
    }

    public List<Column> getColumns() {
        return this.columns;
    }

    public void setColumns(List<Column> columns) {
        this.columns = columns;
    }

    public void addColumn(Column column) {
        this.columns.add(column);
    }
}

```




