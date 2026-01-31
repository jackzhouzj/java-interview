# EasyExcel 完整教程

## 📋 目录
- [技术概述](#技术概述)
- [学习目标](#学习目标)
- [基础概念](#基础概念)
- [核心特性](#核心特性)
- [实战应用](#实战应用)
- [最佳实践](#最佳实践)
- [常见问题](#常见问题)
- [相关资源](#相关资源)
- [学习检查清单](#学习检查清单)

## 📚 技术概述
- **版本**: 3.3.x
- **官方文档**: https://easyexcel.opensource.alibaba.com/
- **GitHub**: https://github.com/alibaba/easyexcel
- **学习难度**: ⭐⭐ (1-5星)
- **重要程度**: ⭐⭐⭐⭐ (1-5星)
- **前置知识**: Java基础、Maven/Gradle、POI基础概念
- **文档来源**: Context7 - alibaba/easyexcel
- **更新时间**: 2024-01-04

### 什么是 EasyExcel

EasyExcel 是阿里巴巴开源的一个基于 Java 的、快速、简洁、解决大文件内存溢出的 Excel 处理工具。它重写了 Apache POI 对 Excel 的解析，能够在极低内存消耗的情况下完成 Excel 的读写操作。

### 核心优势

1. **内存占用低**: 64M 内存可以读取 75M 的 Excel 文件（POI 需要 100M+）
2. **性能优异**: 读写速度比 POI 快 2-3 倍
3. **API 简洁**: 链式调用，代码简洁易懂
4. **功能强大**: 支持注解、模板填充、样式设置等高级功能

## 🎯 学习目标
- [ ] 掌握 EasyExcel 的基本读写操作
- [ ] 理解大文件处理的内存优化原理
- [ ] 掌握注解的使用和自定义转换器
- [ ] 能够处理复杂的 Excel 导入导出场景
- [ ] 掌握模板填充和样式设置
- [ ] 了解 Web 应用中的集成方式

## 📖 基础概念

### 1.1 EasyExcel 与 POI 的区别

| 特性 | Apache POI | EasyExcel |
|------|-----------|-----------|
| 内存占用 | 高（全量加载） | 低（流式处理） |
| 读写速度 | 较慢 | 快 2-3 倍 |
| API 复杂度 | 复杂 | 简洁 |
| 大文件支持 | 容易 OOM | 优秀 |
| 学习成本 | 高 | 低 |

### 1.2 核心组件

- **EasyExcel**: 入口类，提供静态方法创建读写器
- **ReadListener**: 读取监听器，处理每一行数据
- **ExcelWriter**: Excel 写入器，用于大数据量写入
- **WriteSheet**: 工作表对象，表示一个 Sheet
- **@ExcelProperty**: 注解，用于映射 Excel 列

### 1.3 应用场景

- **数据导入**: 用户批量上传数据到系统
- **数据导出**: 系统数据导出为 Excel 报表
- **报表生成**: 基于模板生成复杂报表
- **数据迁移**: 系统间数据迁移
- **数据分析**: 大数据量的 Excel 数据分析

## 🔥 核心特性 (重点)

### 2.1 简单读取 🔥

EasyExcel 提供了多种读取方式，最常用的是监听器模式。

#### 2.1.1 定义实体类

```java
import com.alibaba.excel.annotation.ExcelProperty;
import lombok.Data;
import java.util.Date;

/**
 * Excel 数据实体类
 * @author erik.zhou
 */
@Data
public class DemoData {
    @ExcelProperty("字符串标题")
    private String string;
    
    @ExcelProperty("日期标题")
    private Date date;
    
    @ExcelProperty("数字标题")
    private Double doubleData;
}
```

#### 2.1.2 实现监听器

```java
import com.alibaba.excel.context.AnalysisContext;
import com.alibaba.excel.read.listener.ReadListener;
import com.alibaba.excel.util.ListUtils;
import lombok.extern.slf4j.Slf4j;
import java.util.List;

/**
 * Excel 读取监听器
 * @author erik.zhou
 */
@Slf4j
public class DemoDataListener implements ReadListener<DemoData> {
    
    /**
     * 批处理数量
     */
    private static final int BATCH_COUNT = 100;
    
    /**
     * 缓存的数据列表
     */
    private List<DemoData> cachedDataList = ListUtils.newArrayListWithExpectedSize(BATCH_COUNT);
    
    /**
     * 每解析一行数据都会调用此方法
     */
    @Override
    public void invoke(DemoData data, AnalysisContext context) {
        log.info("解析到一条数据:{}", data);
        cachedDataList.add(data);
        
        // 达到批处理数量，执行存储操作
        if (cachedDataList.size() >= BATCH_COUNT) {
            saveData();
            // 清空缓存列表
            cachedDataList = ListUtils.newArrayListWithExpectedSize(BATCH_COUNT);
        }
    }
    
    /**
     * 所有数据解析完成后调用
     */
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        // 保存剩余数据
        saveData();
        log.info("所有数据解析完成!");
    }
    
    /**
     * 存储数据到数据库
     */
    private void saveData() {
        log.info("{}条数据,开始存储数据库!", cachedDataList.size());
        // 实际业务中这里应该调用 DAO 保存到数据库
        // demoDataMapper.batchInsert(cachedDataList);
        log.info("存储数据库成功!");
    }
}
```

#### 2.1.3 执行读取

```java
import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.read.listener.PageReadListener;

/**
 * Excel 读取示例
 * @author erik.zhou
 */
public class ReadDemo {
    
    /**
     * 方式1: 使用 PageReadListener (推荐，JDK8+)
     */
    public void simpleRead() {
        String fileName = "/path/to/demo.xlsx";
        
        EasyExcel.read(fileName, DemoData.class, new PageReadListener<DemoData>(dataList -> {
            for (DemoData demoData : dataList) {
                log.info("读取到数据: {}", demoData);
            }
        })).sheet().doRead();
    }
    
    /**
     * 方式2: 使用自定义监听器
     */
    public void customListenerRead() {
        String fileName = "/path/to/demo.xlsx";
        
        EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
            .sheet()
            .doRead();
    }
    
    /**
     * 方式3: 同步读取（不推荐，数据量大会 OOM）
     */
    public void syncRead() {
        String fileName = "/path/to/demo.xlsx";
        
        List<DemoData> list = EasyExcel.read(fileName)
            .head(DemoData.class)
            .sheet()
            .doReadSync();
            
        for (DemoData data : list) {
            log.info("读取到数据: {}", data);
        }
    }
}
```

### 2.2 简单写入 🔥

#### 2.2.1 最简单的写入

```java
import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.util.ListUtils;
import java.util.Date;
import java.util.List;

/**
 * Excel 写入示例
 * @author erik.zhou
 */
public class WriteDemo {
    
    /**
     * 准备测试数据
     */
    private List<DemoData> data() {
        List<DemoData> list = ListUtils.newArrayList();
        for (int i = 0; i < 10; i++) {
            DemoData data = new DemoData();
            data.setString("字符串" + i);
            data.setDate(new Date());
            data.setDoubleData(0.56);
            list.add(data);
        }
        return list;
    }
    
    /**
     * 最简单的写入
     */
    public void simpleWrite() {
        String fileName = "/path/to/output" + System.currentTimeMillis() + ".xlsx";
        
        EasyExcel.write(fileName, DemoData.class)
            .sheet("模板")
            .doWrite(data());
    }
}
```

#### 2.2.2 使用 ExcelWriter (推荐大数据量场景) 🔥

```java
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.write.metadata.WriteSheet;

/**
 * 大数据量写入示例
 * @author erik.zhou
 */
public class LargeDataWriteDemo {
    
    /**
     * 使用 ExcelWriter 写入大数据量
     */
    public void largeDataWrite() {
        String fileName = "/path/to/output" + System.currentTimeMillis() + ".xlsx";
        
        // try-with-resources 自动关闭资源
        try (ExcelWriter excelWriter = EasyExcel.write(fileName, DemoData.class).build()) {
            WriteSheet writeSheet = EasyExcel.writerSheet("模板").build();
            
            // 分批写入，每次写入 1000 条
            for (int i = 0; i < 10; i++) {
                // 模拟分页查询数据库
                List<DemoData> dataList = queryDataFromDB(i, 1000);
                excelWriter.write(dataList, writeSheet);
            }
        }
    }
    
    /**
     * 模拟从数据库分页查询数据
     */
    private List<DemoData> queryDataFromDB(int page, int size) {
        // 实际业务中这里应该调用 DAO 分页查询
        return data();
    }
}
```

### 2.3 指定列读取 🔥

使用 `@ExcelProperty` 注解可以按索引或列名指定要读取的列。

```java
import com.alibaba.excel.annotation.ExcelProperty;
import lombok.Data;
import java.util.Date;

/**
 * 指定列读取实体类
 * @author erik.zhou
 */
@Data
public class IndexOrNameData {
    
    /**
     * 按索引指定（从 0 开始）
     */
    @ExcelProperty(index = 0)
    private String string;
    
    /**
     * 按列名指定
     */
    @ExcelProperty("日期")
    private Date date;
    
    /**
     * 按索引指定
     */
    @ExcelProperty(index = 2)
    private Double doubleData;
}
```

### 2.4 大文件处理 (⚠️ 难点)

EasyExcel 对大文件的处理进行了优化，主要通过以下方式：

#### 2.4.1 内存优化原理

1. **流式读取**: 不会一次性将整个文件加载到内存
2. **共享字符串缓存**: 使用文件缓存代替内存缓存
3. **批处理**: 通过监听器批量处理数据

#### 2.4.2 强制使用内存缓存（适用于中等文件 + 低并发）

```java
import com.alibaba.excel.cache.MapCache;

/**
 * 大文件读取优化
 * @author erik.zhou
 */
public class LargeFileReadDemo {
    
    /**
     * 使用内存缓存（提升性能，但内存占用更高）
     */
    public void readWithMemoryCache() {
        String fileName = "/path/to/large-file.xlsx";
        
        EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
            .readCache(new MapCache())  // 强制使用内存缓存
            .sheet()
            .doRead();
    }
}
```

### 2.5 注解详解

#### 2.5.1 @ExcelProperty

```java
import com.alibaba.excel.annotation.ExcelProperty;
import com.alibaba.excel.annotation.write.style.ColumnWidth;
import com.alibaba.excel.annotation.write.style.ContentRowHeight;
import com.alibaba.excel.annotation.write.style.HeadRowHeight;

/**
 * 注解使用示例
 * @author erik.zhou
 */
@Data
@ContentRowHeight(20)  // 内容行高
@HeadRowHeight(30)     // 表头行高
public class AnnotationData {
    
    @ExcelProperty(value = "字符串标题", index = 0)
    @ColumnWidth(20)  // 列宽
    private String string;
    
    @ExcelProperty(value = "日期标题", index = 1)
    @ColumnWidth(15)
    private Date date;
    
    @ExcelProperty(value = "数字标题", index = 2)
    @ColumnWidth(10)
    private Double doubleData;
}
```

## 💻 实战应用

### 3.1 环境搭建

#### 3.1.1 Maven 依赖

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>easyexcel</artifactId>
    <version>3.3.4</version>
</dependency>

<!-- Lombok（可选，简化代码） -->
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.18.30</version>
    <scope>provided</scope>
</dependency>
```

#### 3.1.2 Gradle 依赖

```gradle
implementation 'com.alibaba:easyexcel:3.3.4'
compileOnly 'org.projectlombok:lombok:1.18.30'
annotationProcessor 'org.projectlombok:lombok:1.18.30'
```

### 3.2 Web 应用集成

#### 3.2.1 文件上传（导入）

```java
import com.alibaba.excel.EasyExcel;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

/**
 * Excel 导入导出 Controller
 * @author erik.zhou
 */
@RestController
@RequestMapping("/excel")
public class ExcelController {
    
    /**
     * 文件上传（导入）
     */
    @PostMapping("/upload")
    public Result upload(@RequestParam("file") MultipartFile file) {
        try {
            EasyExcel.read(file.getInputStream(), DemoData.class, new DemoDataListener())
                .sheet()
                .doRead();
            return Result.success("导入成功");
        } catch (Exception e) {
            log.error("导入失败", e);
            return Result.error("导入失败: " + e.getMessage());
        }
    }
}
```

#### 3.2.2 文件下载（导出）

```java
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.net.URLEncoder;

/**
 * Excel 导出示例
 * @author erik.zhou
 */
@RestController
@RequestMapping("/excel")
public class ExcelController {
    
    /**
     * 文件下载（导出）
     */
    @GetMapping("/download")
    public void download(HttpServletResponse response) throws IOException {
        // 设置响应头
        response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
        response.setCharacterEncoding("utf-8");
        
        // 防止中文乱码
        String fileName = URLEncoder.encode("数据导出", "UTF-8").replaceAll("\\+", "%20");
        response.setHeader("Content-disposition", "attachment;filename*=utf-8''" + fileName + ".xlsx");
        
        // 查询数据
        List<DemoData> dataList = queryDataFromDB();
        
        // 写入响应流
        EasyExcel.write(response.getOutputStream(), DemoData.class)
            .sheet("数据")
            .doWrite(dataList);
    }
    
    private List<DemoData> queryDataFromDB() {
        // 实际业务中从数据库查询
        return new ArrayList<>();
    }
}
```

### 3.3 自定义转换器

```java
import com.alibaba.excel.converters.Converter;
import com.alibaba.excel.enums.CellDataTypeEnum;
import com.alibaba.excel.metadata.GlobalConfiguration;
import com.alibaba.excel.metadata.data.ReadCellData;
import com.alibaba.excel.metadata.data.WriteCellData;
import com.alibaba.excel.metadata.property.ExcelContentProperty;

/**
 * 自定义性别转换器
 * @author erik.zhou
 */
public class GenderConverter implements Converter<Integer> {
    
    @Override
    public Class<?> supportJavaTypeKey() {
        return Integer.class;
    }
    
    @Override
    public CellDataTypeEnum supportExcelTypeKey() {
        return CellDataTypeEnum.STRING;
    }
    
    /**
     * 读取时：Excel -> Java
     */
    @Override
    public Integer convertToJavaData(ReadCellData<?> cellData, ExcelContentProperty contentProperty,
                                      GlobalConfiguration globalConfiguration) {
        String stringValue = cellData.getStringValue();
        if ("男".equals(stringValue)) {
            return 1;
        } else if ("女".equals(stringValue)) {
            return 0;
        }
        return null;
    }
    
    /**
     * 写入时：Java -> Excel
     */
    @Override
    public WriteCellData<?> convertToExcelData(Integer value, ExcelContentProperty contentProperty,
                                                 GlobalConfiguration globalConfiguration) {
        if (value == 1) {
            return new WriteCellData<>("男");
        } else if (value == 0) {
            return new WriteCellData<>("女");
        }
        return new WriteCellData<>("");
    }
}

// 使用自定义转换器
@Data
public class UserData {
    @ExcelProperty("姓名")
    private String name;
    
    @ExcelProperty(value = "性别", converter = GenderConverter.class)
    private Integer gender;
}
```

### 3.4 模板填充

```java
import com.alibaba.excel.EasyExcel;
import com.alibaba.excel.ExcelWriter;
import com.alibaba.excel.write.metadata.WriteSheet;
import com.alibaba.excel.write.metadata.fill.FillConfig;
import java.util.HashMap;
import java.util.Map;

/**
 * 模板填充示例
 * @author erik.zhou
 */
public class TemplateFillDemo {
    
    /**
     * 简单填充
     */
    public void simpleFill() {
        String templateFileName = "/path/to/template.xlsx";
        String fileName = "/path/to/output.xlsx";
        
        // 准备填充数据
        Map<String, Object> map = new HashMap<>();
        map.put("name", "张三");
        map.put("age", 25);
        map.put("date", new Date());
        
        // 执行填充
        EasyExcel.write(fileName)
            .withTemplate(templateFileName)
            .sheet()
            .doFill(map);
    }
    
    /**
     * 列表填充
     */
    public void listFill() {
        String templateFileName = "/path/to/template.xlsx";
        String fileName = "/path/to/output.xlsx";
        
        // 准备列表数据
        List<DemoData> dataList = queryDataFromDB();
        
        // 执行填充
        EasyExcel.write(fileName)
            .withTemplate(templateFileName)
            .sheet()
            .doFill(dataList);
    }
}
```

## ✨ 最佳实践

### 4.1 性能优化

#### 4.1.1 读取优化

```java
/**
 * 读取性能优化建议
 * @author erik.zhou
 */
public class ReadOptimization {
    
    /**
     * 1. 使用监听器批处理（推荐）
     */
    public void batchProcess() {
        // 每 100 条批量处理一次，避免内存占用过高
        EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
            .sheet()
            .doRead();
    }
    
    /**
     * 2. 指定读取列，减少数据量
     */
    public void readSpecificColumns() {
        // 只读取需要的列
        EasyExcel.read(fileName, IndexOrNameData.class, new DemoDataListener())
            .sheet()
            .doRead();
    }
    
    /**
     * 3. 跳过表头
     */
    public void skipHeader() {
        EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
            .sheet()
            .headRowNumber(2)  // 跳过前 2 行
            .doRead();
    }
}
```

#### 4.1.2 写入优化

```java
/**
 * 写入性能优化建议
 * @author erik.zhou
 */
public class WriteOptimization {
    
    /**
     * 1. 使用 ExcelWriter 分批写入（推荐大数据量）
     */
    public void batchWrite() {
        try (ExcelWriter excelWriter = EasyExcel.write(fileName, DemoData.class).build()) {
            WriteSheet writeSheet = EasyExcel.writerSheet("数据").build();
            
            // 分批查询，分批写入
            int pageSize = 1000;
            int pageNum = 0;
            List<DemoData> dataList;
            
            do {
                dataList = queryDataFromDB(pageNum++, pageSize);
                if (!dataList.isEmpty()) {
                    excelWriter.write(dataList, writeSheet);
                }
            } while (dataList.size() == pageSize);
        }
    }
    
    /**
     * 2. 关闭自动列宽（提升性能）
     */
    public void disableAutoColumnWidth() {
        EasyExcel.write(fileName, DemoData.class)
            .registerWriteHandler(new LongestMatchColumnWidthStyleStrategy())  // 自定义列宽策略
            .sheet("数据")
            .doWrite(dataList);
    }
}
```

### 4.2 常见陷阱

#### ⚠️ 陷阱 1: 同步读取大文件导致 OOM

```java
// ❌ 错误示例：大文件同步读取会 OOM
List<DemoData> list = EasyExcel.read(fileName)
    .head(DemoData.class)
    .sheet()
    .doReadSync();  // 一次性加载所有数据到内存

// ✅ 正确示例：使用监听器流式处理
EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
    .sheet()
    .doRead();
```

#### ⚠️ 陷阱 2: 监听器中不要做耗时操作

```java
// ❌ 错误示例：每条数据都调用数据库
@Override
public void invoke(DemoData data, AnalysisContext context) {
    demoDataMapper.insert(data);  // 每次插入一条，性能差
}

// ✅ 正确示例：批量处理
@Override
public void invoke(DemoData data, AnalysisContext context) {
    cachedDataList.add(data);
    if (cachedDataList.size() >= BATCH_COUNT) {
        demoDataMapper.batchInsert(cachedDataList);  // 批量插入
        cachedDataList.clear();
    }
}
```

#### ⚠️ 陷阱 3: 忘记关闭 ExcelWriter

```java
// ❌ 错误示例：未关闭资源
ExcelWriter excelWriter = EasyExcel.write(fileName, DemoData.class).build();
WriteSheet writeSheet = EasyExcel.writerSheet("数据").build();
excelWriter.write(dataList, writeSheet);
// 忘记调用 excelWriter.finish()

// ✅ 正确示例：使用 try-with-resources
try (ExcelWriter excelWriter = EasyExcel.write(fileName, DemoData.class).build()) {
    WriteSheet writeSheet = EasyExcel.writerSheet("数据").build();
    excelWriter.write(dataList, writeSheet);
}  // 自动关闭
```

#### ⚠️ 陷阱 4: 模板文件过大

```java
// ⚠️ 注意：模板文件会完全加载到内存
// 模板文件不宜过大（建议 < 10MB）
// 复杂填充建议使用 ExcelWriter 配合 FillConfig
```

### 4.3 异常处理

```java
import com.alibaba.excel.exception.ExcelAnalysisException;

/**
 * 异常处理示例
 * @author erik.zhou
 */
public class ExceptionHandling {
    
    /**
     * 读取时的异常处理
     */
    public void readWithExceptionHandling() {
        try {
            EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
                .sheet()
                .doRead();
        } catch (ExcelAnalysisException e) {
            log.error("Excel 解析异常", e);
            // 处理解析异常
        } catch (Exception e) {
            log.error("读取 Excel 失败", e);
            // 处理其他异常
        }
    }
    
    /**
     * 监听器中的异常处理
     */
    @Slf4j
    public class SafeDemoDataListener implements ReadListener<DemoData> {
        
        @Override
        public void invoke(DemoData data, AnalysisContext context) {
            try {
                // 数据处理逻辑
                processData(data);
            } catch (Exception e) {
                log.error("处理数据异常，行号: {}, 数据: {}", 
                    context.readRowHolder().getRowIndex(), data, e);
                // 记录异常但不中断读取
            }
        }
        
        @Override
        public void doAfterAllAnalysed(AnalysisContext context) {
            log.info("所有数据解析完成");
        }
        
        @Override
        public void onException(Exception exception, AnalysisContext context) {
            log.error("解析异常，行号: {}", context.readRowHolder().getRowIndex(), exception);
            // 可以选择继续或中断
            // throw exception;  // 抛出异常会中断读取
        }
    }
}
```

### 4.4 数据校验

```java
import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

/**
 * 数据校验示例
 * @author erik.zhou
 */
@Data
public class ValidatedData {
    
    @ExcelProperty("姓名")
    @NotBlank(message = "姓名不能为空")
    private String name;
    
    @ExcelProperty("年龄")
    @NotNull(message = "年龄不能为空")
    private Integer age;
    
    @ExcelProperty("邮箱")
    @Email(message = "邮箱格式不正确")
    private String email;
}

/**
 * 校验监听器
 */
@Slf4j
public class ValidatedDataListener implements ReadListener<ValidatedData> {
    
    private final Validator validator = Validation.buildDefaultValidatorFactory().getValidator();
    
    @Override
    public void invoke(ValidatedData data, AnalysisContext context) {
        // 执行校验
        Set<ConstraintViolation<ValidatedData>> violations = validator.validate(data);
        
        if (!violations.isEmpty()) {
            StringBuilder sb = new StringBuilder();
            for (ConstraintViolation<ValidatedData> violation : violations) {
                sb.append(violation.getMessage()).append("; ");
            }
            log.error("数据校验失败，行号: {}, 错误: {}", 
                context.readRowHolder().getRowIndex(), sb.toString());
            return;
        }
        
        // 校验通过，处理数据
        processData(data);
    }
    
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        log.info("所有数据解析完成");
    }
}
```

## ❓ 常见问题

### Q1: EasyExcel 和 POI 如何选择？

**A**: 
- **选择 EasyExcel**: 大文件处理、追求性能、API 简洁性
- **选择 POI**: 需要复杂的 Excel 操作（如公式计算、图表等）、历史项目已使用 POI

### Q2: 如何处理日期格式问题？

**A**:
```java
import com.alibaba.excel.annotation.format.DateTimeFormat;

@Data
public class DateData {
    @ExcelProperty("日期")
    @DateTimeFormat("yyyy-MM-dd HH:mm:ss")
    private Date date;
}
```

### Q3: 如何设置单元格样式？

**A**:
```java
import com.alibaba.excel.annotation.write.style.*;
import com.alibaba.excel.enums.poi.HorizontalAlignmentEnum;

@Data
@HeadStyle(fillForegroundColor = 10)  // 表头背景色
@ContentStyle(horizontalAlignment = HorizontalAlignmentEnum.CENTER)  // 内容居中
public class StyledData {
    @ExcelProperty("姓名")
    @ColumnWidth(20)
    private String name;
    
    @ExcelProperty("金额")
    @NumberFormat("#,##0.00")  // 数字格式
    private Double amount;
}
```

### Q4: 如何读取多个 Sheet？

**A**:
```java
// 方式1: 读取所有 Sheet
EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
    .doReadAll();

// 方式2: 读取指定 Sheet
EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
    .sheet(0)  // 按索引
    .doRead();

EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
    .sheet("Sheet1")  // 按名称
    .doRead();

// 方式3: 读取多个指定 Sheet
EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
    .sheet(0)
    .doRead();
EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
    .sheet(1)
    .doRead();
```

### Q5: 如何处理合并单元格？

**A**:
```java
import com.alibaba.excel.metadata.Head;
import com.alibaba.excel.write.merge.AbstractMergeStrategy;
import org.apache.poi.ss.usermodel.Cell;
import org.apache.poi.ss.usermodel.Sheet;
import org.apache.poi.ss.util.CellRangeAddress;

/**
 * 自定义合并策略
 * @author erik.zhou
 */
public class CustomMergeStrategy extends AbstractMergeStrategy {
    
    @Override
    protected void merge(Sheet sheet, Cell cell, Head head, Integer relativeRowIndex) {
        // 合并第一列的第 2-3 行
        if (cell.getRowIndex() == 1 && cell.getColumnIndex() == 0) {
            CellRangeAddress cellRangeAddress = new CellRangeAddress(1, 2, 0, 0);
            sheet.addMergedRegionUnsafe(cellRangeAddress);
        }
    }
}

// 使用合并策略
EasyExcel.write(fileName, DemoData.class)
    .registerWriteHandler(new CustomMergeStrategy())
    .sheet("数据")
    .doWrite(dataList);
```

### Q6: 如何动态生成表头？

**A**:
```java
import com.alibaba.excel.write.metadata.holder.WriteSheetHolder;
import com.alibaba.excel.write.style.column.AbstractColumnWidthStyleStrategy;
import org.apache.poi.ss.usermodel.Cell;

/**
 * 动态表头示例
 * @author erik.zhou
 */
public class DynamicHeadDemo {
    
    public void dynamicHead() {
        // 动态构建表头
        List<List<String>> head = new ArrayList<>();
        head.add(Arrays.asList("姓名"));
        head.add(Arrays.asList("年龄"));
        head.add(Arrays.asList("地址"));
        
        // 动态构建数据
        List<List<Object>> dataList = new ArrayList<>();
        dataList.add(Arrays.asList("张三", 25, "北京"));
        dataList.add(Arrays.asList("李四", 30, "上海"));
        
        // 写入
        EasyExcel.write(fileName)
            .head(head)
            .sheet("数据")
            .doWrite(dataList);
    }
}
```

### Q7: 如何处理超大文件（100W+ 行）？

**A**:
```java
/**
 * 超大文件处理建议
 * @author erik.zhou
 */
public class VeryLargeFileDemo {
    
    /**
     * 读取超大文件
     */
    public void readVeryLargeFile() {
        // 1. 使用监听器批处理
        // 2. 批处理数量适当增大（如 500-1000）
        // 3. 数据库批量插入使用事务
        // 4. 考虑异步处理
        
        EasyExcel.read(fileName, DemoData.class, new DemoDataListener())
            .readCache(new MapCache())  // 使用内存缓存提升性能
            .sheet()
            .doRead();
    }
    
    /**
     * 写入超大文件
     */
    public void writeVeryLargeFile() {
        try (ExcelWriter excelWriter = EasyExcel.write(fileName, DemoData.class).build()) {
            WriteSheet writeSheet = EasyExcel.writerSheet("数据").build();
            
            // 分批查询，分批写入
            int pageSize = 5000;  // 适当增大批次
            int pageNum = 0;
            List<DemoData> dataList;
            
            do {
                dataList = queryDataFromDB(pageNum++, pageSize);
                if (!dataList.isEmpty()) {
                    excelWriter.write(dataList, writeSheet);
                }
            } while (dataList.size() == pageSize);
        }
    }
}
```

### Q8: 如何实现导入进度提示？

**A**:
```java
/**
 * 导入进度监听器
 * @author erik.zhou
 */
@Slf4j
public class ProgressListener implements ReadListener<DemoData> {
    
    private int totalRows = 0;
    private int processedRows = 0;
    
    @Override
    public void invoke(DemoData data, AnalysisContext context) {
        processedRows++;
        
        // 每处理 100 行输出一次进度
        if (processedRows % 100 == 0) {
            log.info("已处理 {} 行", processedRows);
        }
        
        // 处理数据
        processData(data);
    }
    
    @Override
    public void doAfterAllAnalysed(AnalysisContext context) {
        log.info("导入完成，共处理 {} 行", processedRows);
    }
}
```

## 🔗 相关资源

### 官方资源
- **官方文档**: https://easyexcel.opensource.alibaba.com/
- **GitHub 仓库**: https://github.com/alibaba/easyexcel
- **更新日志**: https://github.com/alibaba/easyexcel/releases

### 推荐文章
- 《EasyExcel 官方文档》
- 《EasyExcel 性能优化实践》
- 《EasyExcel 在大数据量场景下的应用》

### 视频教程
- B站搜索"EasyExcel 教程"
- 阿里云开发者社区视频教程

### 相关技术
- Apache POI: Java 操作 Office 文档的基础库
- Hutool: 提供 ExcelUtil 工具类
- Spring Boot: Web 应用集成

## 📝 学习检查清单

- [ ] 理解 EasyExcel 与 POI 的区别
- [ ] 掌握基本的读写操作
- [ ] 理解监听器的工作原理
- [ ] 掌握 @ExcelProperty 等注解的使用
- [ ] 能够处理大文件（100W+ 行）
- [ ] 掌握 Web 应用中的文件上传下载
- [ ] 了解自定义转换器的实现
- [ ] 掌握模板填充功能
- [ ] 理解性能优化的关键点
- [ ] 能够处理常见的异常情况
- [ ] 掌握数据校验的实现方式
- [ ] 了解样式设置和合并单元格

---

**@author erik.zhou**  
**文档来源**: Context7 - alibaba/easyexcel  
**最后更新**: 2024-01-04
