---
title: PHP初级算法系列 - 2
date: 2017-06-21 14:40:13
tags:
 - PHP
 - 算法
categories:
 - 算法
---

算法功能：返回数组中最小的 `k` 个数。

算法实现1：

```
class BDphp3
{
    public function bd_php3($tmp_array, $k)
    {
        $count = 0;
        $length = count($tmp_array);
        if ($length < $k) {
            return false;
        }
        for ($i = 0; $i < $length; $i++) {
            $min = $i;
            for ($j = $i + 1; $j < $length; $j++) {
                if ($tmp_array[$j] < $tmp_array[$min]) {
                    $min = $j;
                }
            }
            $tmp = $tmp_array[$i];
            $tmp_array[$i] = $tmp_array[$min];
            $tmp_array[$min] = $tmp;
            $count++;
            if ($count > $k) {
                break;
            }
        }
        return array_slice($tmp_array, 0, $k);
    }
}
```

算法实现2：

```
class BDphp3
{
    private function replaceMaxValue($arr, $otherNum)
    {
        $max = $arr[0];
        for ($i = 1; $i < count($arr); $i++) {
            if ($arr[$i] > $max) {
                $max = &$arr[$i];
            }
        }
        if ($otherNum < $max) {
            $max = $otherNum;
        }
        return $arr;
    }

    public function getTop($arr, $topNum)
    {
        $topK = array_slice($arr, 0, $topNum);
        $other = array_slice($arr, $topNum);
        foreach ($other as $value) {
            $topK = $this->replaceMaxValue($topK, $value);
        }
        return $topK;
    }
}
```

算法说明：第一种方式中改进了直接插入排序算法，只是将数组排序 k 次，然后返回数组的前 k 个数。第二种方式将数组分割成两部分，前 k 个数值为第一部分，其余数值为第二部分，对于第二部分的每个数值，如果存在小于第一部分最大值的数值，将第一部分的最大值替换，最后返回第一部分。