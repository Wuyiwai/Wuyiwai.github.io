---
title: 批量合并Update SQL
date: 2019-09-02 15:22:56
tags: MySQL
categories: MySQL
---
### 单条Update SQL
```
UPDATE table SET field = 'value' WHERE other_field = 'other_value';
```

### 多条Update SQL合并;
##### 单值多where更改
```
UPDATE table_name 
SET field = CASE id 
    WHEN 1 THEN 'value'
    WHEN 2 THEN 'value'
    WHEN 3 THEN 'value'
END
WHERE id IN (1,2,3)
```

1. case 后接要比对的条件字段.
2. 如上,即当id=1时,set field = value;当id=2时,set field = value;
3. where的部分不影响代码的执行,但是会提高sql执行的效率,确保sql语句只执行需要执行的行数.
##### 更新多值的
```
UPDATE categories 
SET display_order = CASE id 
    WHEN 1 THEN 3 
    WHEN 2 THEN 4 
    WHEN 3 THEN 5 
END, 
title = CASE id 
    WHEN 1 THEN 'New Title 1'
    WHEN 2 THEN 'New Title 2'
    WHEN 3 THEN 'New Title 3'
END
WHERE id IN (1,2,3)
```
##### 业务中使用
```
$display_order = array( 
  1 => 4, 
  2 => 1, 
  3 => 2, 
  4 => 3, 
  5 => 9, 
  6 => 5, 
  7 => 8, 
  8 => 9 
); 
$ids = implode(',', array_keys($display_order)); 
$sql = "UPDATE categories SET display_order = CASE id "; 
foreach ($display_order as $id => $ordinal) { 
  $sql .= sprintf("WHEN %d THEN %d ", $id, $ordinal); 
} 
$sql .= "END WHERE id IN ($ids)"; 
echo $sql;
```