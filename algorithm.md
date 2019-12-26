* 别人的一个文档：https://ziyi2.github.io/algorithms/

## 用与和异或做加法
原理简述

1. 2个数相加，可以分别把这2个数再拆成2个数，共4个数相加
2. 拆法，拆成用异或相加的为一组，用与(进位)相加的为一组
3. 然后组内相加，又得2个数，重复这个过程

例子
```js
/**
 * 如下两数相加
 * 101101 a = a1 + a2
 *  10101 b = b1 + b2
 * 
 * 拆成
 * 异或组
 * 101000 a1
 *  10000 b1
 * 
 * 与进位组
 *    101 a2
 *    101 b2
 * 
 * 各组内相加得两个新数
 * 111000
 *   1010
 * 
 * 然后重复上面过程
 * /
```

```js
// 循环版
function bitAdd(b, a){
  while(b){
      [b, a] = [(b & a) << 1, b ^ a];
  }
  return a;
}

// 递归版
function sum (a, b) {
  if (b===0) return a;
  return sum(a^b, (a&b)<<1)
}
```

## 排序
```js
// 冒泡排序
function bubbleSort(arr) {
  for (let i = arr.length; i >= 1; i--) {
      for (let j = 0; j < i; j++) {
          if (arr[j] > arr[j + 1]) {
              const temp = arr[j];
              arr[j] = arr[j + 1];
              arr[j + 1] = temp;
          }
      }
  }
  return arr;
}

// 快速排序(inplace)
function quickSort(arr) {
  // 交换
  function swap(arr, a, b) {
      var temp = arr[a];
      arr[a] = arr[b];
      arr[b] = temp;
  }

  // 分区
  function partition(arr, left, right) {
      /**
       * 开始时不知最终pivot的存放位置，可以先将pivot交换到后面去
       * 这里用数组中间的元素为基准
       */
      var pivotIndex = Math.floor((right-left) / 2+left);
      var pivot = arr[pivotIndex];
      /**
       * 存放小于pivot的元素时，是紧挨着上一元素的，否则空隙里存放的可能是大于pivot的元素，
       * 故声明一个storeIndex变量，并初始化为left来依次紧挨着存放小于pivot的元素。
       */
      var storeIndex = left;
      for (var i = left; i <= right; i++) {
          // 注意i<=right，因为pivot是中间的一个量，所以整个区间都要历遍
          if (arr[i] < pivot) {
              /**
               * 遍历数组，找到小于的pivot的元素，（大于pivot的元素会跳过）
               * 将循环i次时得到的元素，通过swap交换放到storeIndex处，
               * 并对storeIndex递增1，表示下一个可能要交换的位置
               */
              swap(arr, storeIndex, i);
              if(pivotIndex == storeIndex) pivotIndex = i; // 如果原本的pivot和某个元素交换了，要记得pivot交换后的位置
              storeIndex++;
          }
      }
      // 最后： 将pivot交换到storeIndex处，基准元素放置到最终正确位置上
      swap(arr, pivotIndex, storeIndex);
      return storeIndex;
  }

  function sort(arr, left, right) {
      if (left >= right) return; // 相等，表示区间长度为1。大于，是因为上一次拿到的值刚好是区间最值，导致分割后少了一个区间

      var storeIndex = partition(arr, left, right);
      sort(arr, left, storeIndex - 1);
      sort(arr, storeIndex + 1, right);
  }

  sort(arr, 0, arr.length - 1);
  return arr;
}

// 归并排序，合并两个有序的子序列
let arr1 = [1,2,6,8];
let arr2 = [1,4,5,7];

function mergeSort(arr1, arr2){
  let res = [];
  let i1 = 0; // 数组1的下标
  let i2 = 0; // 数组2的下标
  while(i1 < arr1.length || i2 < arr2.length){
    if(i1 == arr1.length) {
      res.push(arr2[i2]); // arr1已经遍历完了，所以把arr2剩下的放到后面
      i2++;
    } else if(i2 == arr2.length) {
      res.push(arr1[i1]); // arr2已经遍历完了，所以把arr1剩下的放到后面
      i1++;
    } else if(arr1[i1] < arr2[i2]) {
      res.push(arr1[i1]); // 这里和下面的else，就是谁小就先放谁
      i1++;
    } else {
      res.push(arr2[i2]);
      i2++;
    }
  }
  return res;
}

console.log(mergeSort(arr1, arr2));
```

## KMP

```js
let A = 'abaabaabbabaaabaabbabaab';
let B = 'abaabbabaab';

//找B的最长共同前缀后缀（这里简称公共序列），这里算出来的是减1的，所以后面用的时候要加1
function getF(B){
  let m = B.length;
  let F = new Array(m);
  F[0] = -1; // 第一个肯定是-1
  for (let i=1;i<m;i++) {
      let j=F[i-1];
      // j表示的是公共序列的结束字符在B的位置
      // 下面这段，就是在找之前的一个可用公共序列，并且拼上新增字符，也是公共序列，如果找不到，那就重新开始算
      while (B[j+1]!=B[i] && j>=0) {
        // 当之前一个B的片段有公共序列，但是把B[i]算进去时，并不能形成公共序列时，就要找次长公共序列
        // 找次长公共序列，其实就是找B[0:j]这段的公共序列
        // 如果B[0:j]没有公共序列，也就是F[j]==-1，就退出循环，这时公共序列不能增长了，会重新算
        // 如果找到B[j+1]==B[i]，则有B[0:j+1]==B[i-j-1:i]，则F[i]=j+1，在下面的判断里执行
        j = F[j];
      }

      if (B[j+1]==B[i]) {
        F[i]=j+1;
      } else {
        F[i]=-1;
      }
  }
  return F;
}

function kmp(A, B){
  let n = A.length;
  let m = B.length;
  let F = getF(B);

  let i = 0; // 用来表示B开头对准A的位置，如果匹配成功，这个就是最终要返回的结果
  let j = 0; // 用来表示B和A已经匹配的长度
  let tem;

  while(i<n){ // 这里肯定不能用for了，因为会跳
    if(A[i+j] == B[j]) {
      j++;
      if(j == m) {
        console.log(i); // 把找到的都输出
        tem = F[j-1]+1; // 这里同样走一次根据公共序列移动B的流程
        i = i + (j-tem);
        j = tem;
      }
    } else {
      if(j == 0) {
        // 一个都不匹配，b直接后移一位
        i++;
      } else {
        // 有部分匹配了
        tem = F[j-1]+1; // 已经匹配的部分，公共前缀后缀的长度
        // 要把B往后移动，让B开头对准已经匹配的部分的后缀（就是让已经匹配的部分的公共前缀（在B）后缀（在A）对准）
        i = i + (j-tem); // 要移动的位数，就是已经匹配的长度j，减去公共前缀的长度
        j = tem; // 并且要重置已经匹配的长度
      }
    }
  }
}
```