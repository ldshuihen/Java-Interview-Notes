### 数组

以下是几道Java数组相关的代码面试题及答案，涵盖了数组的常见操作和特性：

#### 题目1：数组反转

编写一个方法，将给定的int数组反转。

示例：输入[1,2,3,4,5]，输出[5,4,3,2,1]

答案：
```java
public class ArrayReverse {
    public static void reverseArray(int[] arr) {
        if (arr == null || arr.length <= 1) {
            return;
        }
        
        int left = 0;
        int right = arr.length - 1;
        
        while (left < right) {
            // 交换元素
            int temp = arr[left];
            arr[left] = arr[right];
            arr[right] = temp;
            
            left++;
            right--;
        }
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 4, 5};
        reverseArray(arr);
        
        for (int num : arr) {
            System.out.print(num + " "); // 输出: 5 4 3 2 1
        }
    }
}
```

#### 题目2：找出数组中的重复元素

编写一个方法，找出int数组中重复出现的元素。

示例：输入[1,2,3,2,1,5,6,5,5,5]，输出[1,2,5]

答案：
```java
import java.util.*;

public class FindDuplicates {
    public static List<Integer> findDuplicates(int[] arr) {
        Set<Integer> seen = new HashSet<>();
        Set<Integer> duplicates = new HashSet<>();
        
        for (int num : arr) {
            if (!seen.add(num)) {
                duplicates.add(num);
            }
        }
        
        return new ArrayList<>(duplicates);
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 2, 1, 5, 6, 5, 5, 5};
        List<Integer> duplicates = findDuplicates(arr);
        
        System.out.println(duplicates); // 输出: [1, 2, 5]
    }
}
```

#### 题目3：找出数组中出现次数最多的元素

编写一个方法，找出int数组中出现次数最多的元素。

示例：输入[1,2,3,2,1,5,6,5,5,5]，输出5

答案：
```java
import java.util.HashMap;
import java.util.Map;

public class MostFrequentElement {
    public static int findMostFrequent(int[] arr) {
        if (arr == null || arr.length == 0) {
            throw new IllegalArgumentException("数组不能为空");
        }
        
        Map<Integer, Integer> countMap = new HashMap<>();
        
        // 统计每个元素出现的次数
        for (int num : arr) {
            countMap.put(num, countMap.getOrDefault(num, 0) + 1);
        }
        
        // 找出出现次数最多的元素
        int maxCount = 0;
        int mostFrequent = arr[0];
        
        for (Map.Entry<Integer, Integer> entry : countMap.entrySet()) {
            if (entry.getValue() > maxCount) {
                maxCount = entry.getValue();
                mostFrequent = entry.getKey();
            }
        }
        
        return mostFrequent;
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 2, 1, 5, 6, 5, 5, 5};
        System.out.println(findMostFrequent(arr)); // 输出: 5
    }
}
```

#### 题目4：合并两个有序数组

编写一个方法，将两个有序int数组合并成一个新的有序数组。

示例：输入[1,3,5]和[2,4,6]，输出[1,2,3,4,5,6]

答案：
```java
public class MergeSortedArrays {
    public static int[] mergeArrays(int[] arr1, int[] arr2) {
        if (arr1 == null) return arr2;
        if (arr2 == null) return arr1;
        
        int[] result = new int[arr1.length + arr2.length];
        int i = 0, j = 0, k = 0;
        
        // 比较两个数组的元素，按顺序放入结果数组
        while (i < arr1.length && j < arr2.length) {
            if (arr1[i] <= arr2[j]) {
                result[k++] = arr1[i++];
            } else {
                result[k++] = arr2[j++];
            }
        }
        
        // 处理剩余元素
        while (i < arr1.length) {
            result[k++] = arr1[i++];
        }
        
        while (j < arr2.length) {
            result[k++] = arr2[j++];
        }
        
        return result;
    }
    
    public static void main(String[] args) {
        int[] arr1 = {1, 3, 5};
        int[] arr2 = {2, 4, 6};
        int[] merged = mergeArrays(arr1, arr2);
        
        for (int num : merged) {
            System.out.print(num + " "); // 输出: 1 2 3 4 5 6
        }
    }
}
```

#### 题目5：找出数组中只出现一次的元素

编写一个方法，找出int数组中只出现一次的元素（假设数组中除了一个元素只出现一次外，其他元素都出现两次）。

示例：输入[1,2,3,2,1]，输出3

答案：
```java
public class SingleNumber {
    public static int findSingleNumber(int[] arr) {
        int result = 0;
        // 利用异或运算的特性：a ^ a = 0，a ^ 0 = a
        for (int num : arr) {
            result ^= num;
        }
        return result;
    }
    
    public static void main(String[] args) {
        int[] arr = {1, 2, 3, 2, 1};
        System.out.println(findSingleNumber(arr)); // 输出: 3
    }
}
```

这些题目涵盖了数组的基本操作、常见算法和Java集合框架的使用，考察了对数组特性和常用处理方式的理解。



### 集合框架

以下是几道 Java 集合框架的代码测试题及参考答案，涵盖了 List、Set、Map 等核心组件的常见操作场景：

#### 题1：List 元素去重（保持顺序）

**需求**：编写方法对 List 去重并保持元素首次出现顺序

```java
import java.util.ArrayList;
import java.util.LinkedHashSet;
import java.util.List;

public class ListDeduplication {
    // 方法1：利用LinkedHashSet的特性（去重且保持插入顺序）
    public static List<Integer> removeDuplicates(List<Integer> list) {
        if (list == null || list.isEmpty()) {
            return new ArrayList<>();
        }
        // LinkedHashSet会保留元素插入顺序，同时去重
        LinkedHashSet<Integer> set = new LinkedHashSet<>(list);
        return new ArrayList<>(set);
    }

    public static void main(String[] args) {
        List<Integer> input = List.of(3, 1, 2, 3, 1, 4, 2);
        List<Integer> result = removeDuplicates(input);
        System.out.println(result); // 输出: [3, 1, 2, 4]
    }
}

```





**解析**：  
使用 `LinkedHashSet` 兼具 `HashSet` 的去重功能和 `LinkedList` 的顺序保留特性，是最简洁的实现方式。

#### 题2：统计字符串中字符出现次数

**需求**：使用 Map 统计字符出现次数并按字符升序输出

```java
import java.util.*;

public class CharCount {
    public static void countCharacters(String str) {
        if (str == null || str.isEmpty()) {
            return;
        }
        
        // 统计字符出现次数
        Map<Character, Integer> countMap = new HashMap<>();
        for (char c : str.toCharArray()) {
            countMap.put(c, countMap.getOrDefault(c, 0) + 1);
        }
        
        
        // 将键按自然顺序排序
        List<Character> sortedChars = new ArrayList<>(countMap.keySet());
        Collections.sort(sortedChars);
        
        // 输出结果
        for (char c : sortedChars) {
            System.out.println(c + ":" + countMap.get(c));
        }
    }

    public static void main(String[] args) {
        countCharacters("HelloWorld");
        // 输出:
        // H:1
        // W:1
        // d:1
        // e:1
        // l:3
        // o:2
        // r:1
    }
}

```



**解析**：  
1. 使用 `HashMap` 存储字符与出现次数的映射  
2. 通过 `getOrDefault()` 简化计数逻辑  
3. 利用 `Collections.sort()` 对键进行自然排序

#### 题3：找出两个集合的交集

**需求**：实现两个 Set 的交集计算



```java
import java.util.HashSet;
import java.util.Set;

public class SetIntersection {
    public static Set<String> findIntersection(Set<String> set1, Set<String> set2) {
        // 避免修改原集合，创建新集合进行操作
        Set<String> result = new HashSet<>(set1);
        // 保留同时存在于两个集合中的元素
        result.retainAll(set2);
        return result;
    }

    public static void main(String[] args) {
        Set<String> set1 = new HashSet<>(Set.of("a", "b", "c"));
        Set<String> set2 = new HashSet<>(Set.of("b", "c", "d"));
        Set<String> intersection = findIntersection(set1, set2);
        System.out.println(intersection); // 输出: [b, c]
    }
}

```



**解析**：  
利用 `Set.retainAll()` 方法直接保留两个集合的共有元素，这是集合交集计算的标准实现方式。

#### 题4：Map 按值排序

**需求**：将 Map 按值（年龄）升序排序

```java
import java.util.*;

public class MapSortByValue {
    public static List<Map.Entry<String, Integer>> sortByValue(Map<String, Integer> map) {
        // 将entrySet转换为List以便排序
        List<Map.Entry<String, Integer>> entryList = new ArrayList<>(map.entrySet());
        
        // 按值升序排序
        entryList.sort(Comparator.comparingInt(Map.Entry::getValue));
        
        return entryList;
    }

    public static void main(String[] args) {
        Map<String, Integer> ageMap = new HashMap<>();
        ageMap.put("张三", 25);
        ageMap.put("李四", 20);
        ageMap.put("王五", 30);
        
        List<Map.Entry<String, Integer>> sortedList = sortByValue(ageMap);
        System.out.println(sortedList); // 输出: [李四=20, 张三=25, 王五=30]
    }
}

```



注意：**`Map.Entry`**：是 `Map` 接口中的内部接口，表示 Map 中的一个键值对（Key-Value 对），包含 `getKey()` 和 `getValue()` 方法。

**解析**：  
1. 将 `Map.Entry` 转换为 List 才能进行排序  
2. 使用 `Comparator.comparingInt()` 简化排序逻辑  
3. 若需降序可追加 `.reversed()`


这些题目覆盖了集合框架的核心操作，答案注重利用 Java 集合的内置方法实现，既简洁又符合最佳实践。实际开发中，应优先使用 JDK 提供的工具方法而非手动实现复杂逻辑。

### IO/NIO

以下是几道 Java IO/NIO 的代码测试题及参考答案，涵盖了传统 IO 流操作和 NIO 的核心特性：

#### 题1：文件复制（传统 IO 字节流实现）

**需求**：使用字节流复制任意类型文件，支持大文件高效复制

 ```java
package io_nio;

import java.io.*;

/**
 * 文件名：FileCopyWithIO
 * 作者：24141
 * 创建日期：2025/10/15
 * 描述：使用字节流复制任意类型文件，支持大文件高效复制
 */
public class FileCopyWithIO {
    public static final int BUFFER_SIZE = 8 * 1024;
    public static void copyFile(String srcFile, String destFile) throws IOException {
        // try-with-resources 自动关闭资源
        try(FileInputStream fis = new FileInputStream(srcFile);
            FileOutputStream fos = new FileOutputStream(destFile);
            BufferedInputStream bis = new BufferedInputStream(fis);
            BufferedOutputStream bos = new BufferedOutputStream(fos);){
                byte[] buffer = new byte[BUFFER_SIZE];
                int bytesRead;
                // 读取数据到缓冲区，返回 -1 表示读取完毕
                while ((bytesRead = bis.read(buffer)) != -1) {
                    // 写入缓冲区的数据到目标文件
                    bos.write(buffer, 0, bytesRead);
                    // 刷新缓冲区，确保数据写入目标文件
                    bos.flush();
                }
        } catch (Exception e) {
            e.printStackTrace();
        }

    }
    public static void main(String[] args) {
        String sourceFile = "src/main/java/io_nio/assets/lion.png";
        String targetFile = "src/main/java/io_nio/assets/target.png";
        try{
            Long start = System.currentTimeMillis();
            copyFile(sourceFile,targetFile);
            Long end = System.currentTimeMillis();
            System.out.println("复制完成，耗时：" + (end - start) + "ms");
        } catch (IOException e) {
            throw new RuntimeException(e);
        }
    }
}

 ```






**解析**：  
1. 使用 `BufferedInputStream` 和 `BufferedOutputStream` 提供缓冲功能，减少 IO 次数  
2. 设定合理的缓冲区大小（8KB）平衡内存占用和效率  
3. `try-with-resources` 语法自动释放资源，避免内存泄漏  

#### 题2：文本文件按行读取与行数统计

**需求**：读取 UTF-8 编码的文本文件，按行输出内容并统计总行数

 ```java
import java.io.*;
import java.nio.charset.StandardCharsets;

public class TextFileLineReader {
    public static int readAndCountLines(String filePath) throws IOException {
        int lineCount = 0;
        
        // 指定UTF-8编码读取文件，读取字符时：先用字节流->然后转化字符流
        try (FileInputStream fis = new FileInputStream(filePath);
             InputStreamReader isr = new InputStreamReader(fis, StandardCharsets.UTF_8);
             BufferedReader br = new BufferedReader(isr)) {

            String line;
            // 逐行读取，null表示文件结束
            while ((line = br.readLine()) != null) {
                lineCount++;
                System.out.println("第" + lineCount + "行：" + line);
            }
        }
        return lineCount;
    }

    public static void main(String[] args) {
        String filePath = "test.txt";  // 文本文件路径
        try {
            int count = readAndCountLines(filePath);
            System.out.println("文件总行数：" + count);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
    
 ```




**解析**：  
1. `InputStreamReader` 指定 UTF-8 编码，解决中文乱码问题  
2. `BufferedReader` 的 `readLine()` 方法高效实现按行读取  
3. 循环中完成行数统计和内容输出  

#### 题3：NIO 实现文件内容查找

**需求**：使用 NIO API 查找目录下所有包含关键字的文本文件

 ```java
import java.io.IOException;
import java.nio.file.*;
import java.nio.file.attribute.BasicFileAttributes;
import java.util.List;

public class FileContentSearch {
    private final String keyword;
    
    public FileContentSearch(String keyword) {
        this.keyword = keyword;
    }
    
    // 遍历目录下所有文件
    public void searchInDirectory(String dirPath) throws IOException {
        Path rootPath = Paths.get(dirPath);
        
        // 递归遍历目录
        Files.walkFileTree(rootPath, new SimpleFileVisitor<Path>() {
            @Override
            public FileVisitResult visitFile(Path file, BasicFileAttributes attrs) throws IOException {
                // 只处理文本文件（简单判断扩展名）
                if (file.getFileName().toString().endsWith(".txt")) {
                    checkFileContent(file);
                }
                return FileVisitResult.CONTINUE;
            }
            
            // 忽略目录访问错误
            @Override
            public FileVisitResult visitFileFailed(Path file, IOException exc) {
                return FileVisitResult.CONTINUE;
            }
        });
    }
    
    // 检查文件内容是否包含关键字
    private void checkFileContent(Path file) throws IOException {
        List<String> lines = Files.readAllLines(file);
        for (int i = 0; i < lines.size(); i++) {
            if (lines.get(i).contains(keyword)) {
                System.out.printf("文件：%s，第%d行包含关键字：%s%n", 
                        file.getFileName(), i+1, lines.get(i));
            }
        }
    }
    
    public static void main(String[] args) {
        String dir = "documents";    // 目标目录
        String keyword = "Java";     // 要查找的关键字
        
        try {
            new FileContentSearch(keyword).searchInDirectory(dir);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
    
 ```



**解析**：  

1. `Files.walkFileTree()` 递归遍历目录结构  
2. `SimpleFileVisitor` 简化文件访问逻辑，只处理 `.txt` 文件  
3. `Files.readAllLines()` 一次性读取文件内容（适合中小文件）  

#### 题4：NIO 缓冲区实现字符串反转

**需求**：使用 ByteBuffer 实现字符串反转功能

 ```java
import java.nio.ByteBuffer;
import java.nio.charset.StandardCharsets;

public class StringReverseWithNIO {
    public static String reverseString(String input) {
        // 1. 字符串转换为字节数组并写入缓冲区
        byte[] bytes = input.getBytes(StandardCharsets.UTF_8);
        ByteBuffer buffer = ByteBuffer.allocate(bytes.length);
        buffer.put(bytes);
        
        // 2. 切换为读模式
        buffer.flip();
        
        // 3. 从缓冲区读取字节并反转
        byte[] reversedBytes = new byte[bytes.length];
        for (int i = 0; i < bytes.length; i++) {
            // 从缓冲区末尾开始读取
            reversedBytes[i] = buffer.get(bytes.length - 1 - i);
        }
        
        // 4. 转换为字符串返回
        return new String(reversedBytes, StandardCharsets.UTF_8);
    }
    
    public static void main(String[] args) {
        String input = "Hello NIO";
        String reversed = reverseString(input);
        System.out.println("原始字符串：" + input);
        System.out.println("反转后：" + reversed);  // 输出：OIN olleH
    }
}
    
 ```




**解析**：  
1. `ByteBuffer` 用于存储字符串的字节数据  
2. `flip()` 方法切换缓冲区为读模式  
3. 通过反向索引读取实现字符串反转  


这些题目覆盖了 IO/NIO 的核心操作，答案注重资源管理和效率优化。传统 IO 适合简单场景，而 NIO 的通道、缓冲区和文件工具类在复杂操作中更具优势。



### Stream API

下面通过6个典型问题示例，展示Stream API在不同场景下的应用，涵盖数据过滤、转换、聚合、分组等常见操作，每个示例包含问题描述、传统解法与Stream解法的对比：

#### 示例1：筛选并收集符合条件的元素

**问题**：从员工列表中筛选出"部门为研发部且薪资>15000"的员工，收集为新列表。

​    

```java
import java.util.ArrayList;
import java.util.List;
import java.util.stream.Collectors;

// 员工类
class Employee {
    private String name;
    private String department;
    private double salary;

    public Employee(String name, String department, double salary) {
        this.name = name;
        this.department = department;
        this.salary = salary;
    }

    // getter方法
    public String getDepartment() { return department; }
    public double getSalary() { return salary; }
    public String getName() { return name; }

    @Override
    public String toString() {
        return name + "(" + department + ", " + salary + ")";
    }
}

public class StreamFilterDemo {
    public static void main(String[] args) {
        List<Employee> employees = List.of(
            new Employee("张三", "研发部", 20000),
            new Employee("李四", "财务部", 12000),
            new Employee("王五", "研发部", 16000),
            new Employee("赵六", "研发部", 14000)
        );

        // 传统解法：手动循环筛选
        List<Employee> result1 = new ArrayList<>();
        for (Employee emp : employees) {
            if ("研发部".equals(emp.getDepartment()) && emp.getSalary() > 15000) {
                result1.add(emp);
            }
        }

        // Stream解法：链式调用筛选+收集
        List<Employee> result2 = employees.stream()
                .filter(emp -> "研发部".equals(emp.getDepartment()) && emp.getSalary() > 15000)
                .collect(Collectors.toList());

        System.out.println("筛选结果：" + result2);
        // 输出：[张三(研发部, 20000.0), 王五(研发部, 16000.0)]
    }
}
    
```





#### 示例2：转换元素类型并去重

**问题**：将字符串列表中所有元素转为大写后，去除重复值并收集为Set。

​    

```java
import java.util.List;
import java.util.Set;
import java.util.stream.Collectors;

public class StreamMapDistinctDemo {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "Banana", "APPLE", "orange", "banana");

        // 传统解法：手动循环转换+去重
        Set<String> result1 = new java.util.HashSet<>();
        for (String word : words) {
            result1.add(word.toUpperCase());
        }

        // Stream解法：map转换+distinct去重+收集为Set
        Set<String> result2 = words.stream()
                .map(String::toUpperCase)  // 方法引用简化：转为大写
                .distinct()                // 去重（基于equals()）
                .collect(Collectors.toSet());

        System.out.println("处理结果：" + result2);
        // 输出：[APPLE, BANANA, ORANGE]
    }
}
    
```





#### 示例3：聚合计算（求和/平均值）

**问题**：计算所有学生的平均成绩，保留2位小数。

​    

```java
import java.text.DecimalFormat;
import java.util.List;

// 学生类
class Student {
    private String name;
    private double score;

    public Student(String name, double score) {
        this.name = name;
        this.score = score;
    }

    public double getScore() { return score; }
}

public class StreamAverageDemo {
    public static void main(String[] args) {
        List<Student> students = List.of(
            new Student("小明", 88.5),
            new Student("小红", 92.0),
            new Student("小刚", 76.5)
        );

        // 传统解法：手动累加求平均
        double sum = 0;
        for (Student s : students) {
            sum += s.getScore();
        }
        double avg1 = sum / students.size();

        // Stream解法：mapToDouble转换为数值流+average()求平均
        double avg2 = students.stream()
                .mapToDouble(Student::getScore)  // 转换为DoubleStream
                .average()                       // 返回OptionalDouble（避免空集合）
                .orElse(0.0);                    // 空集合时返回0.0

        // 格式化输出（保留2位小数）
        DecimalFormat df = new DecimalFormat("#.00");
        System.out.println("平均成绩：" + df.format(avg2));  // 输出：85.67
    }
}
    
```





#### 示例4：分组统计

**问题**：按部门对员工进行分组，并统计每个部门的人数。

​    

```java
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

// 复用示例1的Employee类
public class StreamGroupDemo {
    public static void main(String[] args) {
        List<Employee> employees = List.of(
            new Employee("张三", "研发部", 20000),
            new Employee("李四", "财务部", 12000),
            new Employee("王五", "研发部", 16000),
            new Employee("赵六", "人事部", 10000)
        );

        // 传统解法：手动遍历分组统计
        Map<String, Integer> count1 = new java.util.HashMap<>();
        for (Employee emp : employees) {
            String dept = emp.getDepartment();
            count1.put(dept, count1.getOrDefault(dept, 0) + 1);
        }

        // Stream解法：Collectors.groupingBy分组+Collectors.counting统计
        Map<String, Long> count2 = employees.stream()
                .collect(Collectors.groupingBy(
                    Employee::getDepartment,  // 分组依据：部门
                    Collectors.counting()     // 统计每组数量（返回Long）
                ));

        System.out.println("部门人数统计：" + count2);
        // 输出：{研发部=2, 财务部=1, 人事部=1}
    }
}
    
```





#### 示例5：查找匹配元素

**问题**：判断列表中是否存在长度大于5的字符串，若存在则返回第一个符合条件的字符串。

​    

```java
import java.util.List;
import java.util.Optional;

public class StreamFindDemo {
    public static void main(String[] args) {
        List<String> words = List.of("apple", "cat", "banana", "dog", "grapefruit");

        // 传统解法：循环查找
        String result1 = null;
        for (String word : words) {
            if (word.length() > 5) {
                result1 = word;
                break;  // 找到第一个就退出
            }
        }

        // Stream解法：anyMatch判断是否存在 + findFirst获取第一个
        boolean exists = words.stream().anyMatch(word -> word.length() > 5);
        Optional<String> firstLongWord = words.stream()
                .filter(word -> word.length() > 5)
                .findFirst();  // 返回Optional，避免空指针

        String result2 = firstLongWord.orElse("无符合条件的字符串");

        System.out.println("是否存在长字符串：" + exists);  // 输出：true
        System.out.println("第一个长字符串：" + result2);   // 输出：banana
    }
}
    
```





#### 示例6：并行流处理大数据

**问题**：计算1到1000万之间所有偶数的和（使用并行流提高效率）。


​    

```java
import java.util.stream.LongStream;

public class StreamParallelDemo {
    public static void main(String[] args) {
        // 传统解法：普通循环（单线程）
        long start1 = System.currentTimeMillis();
        long sum1 = 0;
        for (long i = 1; i <= 10_000_000; i++) {
            if (i % 2 == 0) {
                sum1 += i;
            }
        }
        long time1 = System.currentTimeMillis() - start1;

        // Stream解法：并行流（多线程自动处理）
        long start2 = System.currentTimeMillis();
        long sum2 = LongStream.rangeClosed(1, 10_000_000)  // 生成1到1000万的LongStream
                .parallel()                               // 开启并行处理
                .filter(i -> i % 2 == 0)                  // 筛选偶数
                .sum();                                   // 求和
        long time2 = System.currentTimeMillis() - start2;

        System.out.println("普通循环结果：" + sum1 + "，耗时：" + time1 + "ms");
        System.out.println("并行流结果：" + sum2 + "，耗时：" + time2 + "ms");
        // 输出示例：并行流耗时通常更短（数据量越大优势越明显）
    }
}
    
```



#### 总结：Stream API的核心优势场景

1. **数据筛选与转换**：`filter`+`map`组合替代循环判断与类型转换；
2. **聚合计算**：`sum`/`average`/`count`等直接实现统计需求；
3. **分组与分区**：`Collectors.groupingBy`轻松实现按条件分组；
4. **查找与匹配**：`anyMatch`/`findFirst`等避免手动循环查找；
5. **并行处理**：`parallel()`一行代码实现多线程加速大数据计算。

通过这些示例可以看出，Stream API将数据处理逻辑从"怎么做"（循环、临时变量）转变为"做什么"（声明式操作），代码更简洁、可读性更强，尤其适合处理集合数据时使用。