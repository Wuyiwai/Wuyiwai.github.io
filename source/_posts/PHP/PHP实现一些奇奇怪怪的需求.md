---
title: PHP实现一些奇奇怪怪的需求
date: 2019-10-22 21:25:42
tags: PHP
categories: PHP
---
1. 字符串过滤除ASCII码之外的字符,使用场景:数据库字符集设置为ASCII,而数据来源有其他字符集的数据时
```
preg_replace('/[^(\x20-\x7F)]*/','', trim(strtolower($email)))
```
2. 根据多项key对多维数组进行排序.类似order by 多项
```
/**
     * 根据多项key对多维数组进行排序.类似order by 多项
     * @param array $arr
     * @param array $sortOption
     * @return array|mixed
     * $sortOption = [
     *      'unread_count' => SORT_DESC,
     *      'receive_time' => SORT_DESC,
     * ];
     * $arr = [
     *      [
     *          'receive_time' => 0,
     *          'unread_count' => 200,
     *      ],
     * ];
     */
    public static function sortByManyFields(array $arr = [], array $sortOption = [])
    {
        if (empty($arr) || empty($sortOption)) return [];
        $data = [];
        foreach ($arr as $index => $row) {
            foreach ($sortOption as $key => $sortOrder) {
                $data[$key][$index] = $row[$key];
            }
        }
        $args = [];
        foreach ($sortOption as $key => $sortOrder) {
            $args[] = $data[$key];
            $args[] = $sortOrder;
        }
        $args[] = &$arr;
        call_user_func_array('array_multisort', $args);
        return array_pop($args);
    }
```