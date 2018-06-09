---
title: PHP初级算法系列 - 1
date: 2017-06-21 11:18:56
tags:
 - PHP
 - 算法
categories:
 - 算法
---

给定一个数组：

```
$arr = array('b' => 'a', 'c' => 'a', 'e' => 'b', 'd' => 'b', 'f' => 'c', 'g' => 'e', 'h' => 'f');
```

给出可以完成以下格式转换的算法。

```
Array
(
    [a] => Array
        (
            [b] => Array
                (
                    [e] => Array
                        (
                            [0] => g
                        )

                    [0] => d
                )

            [c] => Array
                (
                    [f] => Array
                        (
                            [0] => h
                        )

                )

        )

)
```

算法实现：

```
class BDphp1
{
    private function php1($tmp_array, $start)
    {
        $result = array();
        if ($ret = array_keys($tmp_array, $start)) {
            foreach ($ret as $v) {
                if ($rev = $this->php1($tmp_array, $v)) {
                    $result[$v] = $rev;
                } else {
                    $result[] = $v;
                }
            }
        }
        return $result;
    }

    public function bd_php1($tmp_array, $start = '')
    {
        $tmp = array();
        if ($start == null) {
            $start = array_shift(array_values($tmp_array));
        }
        $ret = $this->php1($tmp_array, $start);
        $tmp[$start] = $ret;
        return $tmp;
    }

}
```

算法说明：主要使用 `array_shift(),array_values(),array_keys()` 函数和递归思想。  