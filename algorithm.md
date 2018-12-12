* 金山 WPS 算法题
  1. 给定一个数组，给定一个值，要求数组内如果有任意2个数相加等于给定值便返回 true，否则返回 false，要求复杂度为 O(n) <br>
    （自己 2 分钟想到写的答案）利用对象存储需要的差值，除第一次外每次循环进行询问是否存在差值

    ```
    var arr = [10, 9, 8, 4, 7, 11, 7, 2, 5, 10]
    var target = 9

    var fn = function (array, target) {
      var obj = {}
      for (var i = 0; i < arr.length; i++) {
        if (i !== 0 && obj[arr[i]]) return true
        obj[target - arr[i]] = true
      }
      return false
    }
    console.log(fn(arr, target))
    ```
