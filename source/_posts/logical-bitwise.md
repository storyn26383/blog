---
title: 利用位元運算讓程式更簡潔
date: 2021-04-22 23:26:02
tags:
  - Algorithm
  - Bitwise
  - JavaScript
---

邏輯運算我們平時都用得多，但位元運算似乎就顯少使用。
其實位元運算在程式設計中扮演著非常重要的角色，他不僅可以讓程式更簡潔，效能也比較好呢！

<!--more-->

## 位元運算子

先讓我們復習一下位元運算子：

```javascript
// AND
0 & 0 = 0
0 & 1 = 0
1 & 0 = 0
1 & 1 = 1

// OR
0 | 0 = 0
0 | 1 = 1
1 | 0 = 1
1 | 1 = 1

// XOR
0 ^ 0 = 0
0 ^ 1 = 1
1 ^ 0 = 1
1 ^ 1 = 0

// NOT
!0 = 1
!1 = 0
```

複習完了之後，就可以來實戰了！

註：以下範例利用 JavaScript「0 為 false；1 為 true」的特性，讓程式碼更簡潔，但並非所有程式語言都適用。

## 奇偶數判斷

### 一般作法

一般我們都會用 `%`（取餘數）的方式來判斷奇偶數。

```javascript
const isEven = (n) => n % 2

isEven(3) // 0
```

### 位元運算

因為奇數轉換成 2 進位之後最後一位一定會是 1，因此我們就可以用 AND 來簡單判斷是否為奇數了。

```javascript
const isOdd = (n) => n & 1

isOdd(3) // 1
```

## 0 / 1 切換（Toggle）

### 一般作法

```javascript
let show = 0

show = !show // true
show = !show // false
```

### 位元運算

利用 XOR 相同為 0 相異為 1 的特性，因此可以簡單 0 / 1 切換。

```javascript
let show = 0

show ^= 1 // 1
show ^= 1 // 0
```

## 進階應用 - 交集 / 聯集

### 需求說明

每個學生可以選修多個課程，要設計一個功能，輸入課程，並列出有選修該課程的學生們。

### 一般作法

想到多個課程，那當然就是用陣列啦，但在判斷聯集的時候就會非常麻煩，如果量一大，效能也會成為一大問題。

```javascript
const ENGLISH = 1
const MATHEMATICS = 2
const PHYSICAL = 3
const CHEMICAL = 4

const students = [
  { courses: [ENGLISH] },
  { courses: [MATHEMATICS] },
  { courses: [ENGLISH, PHYSICAL] },
  { courses: [MATHEMATICS, CHEMICAL] },
  { courses: [ENGLISH, PHYSICAL, CHEMICAL] }
]

const findStudentsViaCourses = (courses) =>
  return students.filter((student) => {
    for (const course of courses) {
      if (!stuudents.courses.includes(course)) {
        return false
      }
    }

    return true
  })
}

findStudentsViaCourses([ENGLISH, PHYSICAL])
// [
//   { courses: [ENGLISH, PHYSICAL] },
//   { courses: [ENGLISH, PHYSICAL, CHEMICAL] }
// ]
```

### 位元運算

先看結果

```javascript
const ENGLISH = 0b0001 // 1
const MATHEMATICS = 0b0010 // 2
const PHYSICAL = 0b0100 // 4
const CHEMICAL = 0b1000 // 8

const students = [
  { courses: ENGLISH },
  { courses: MATHEMATICS },
  { courses: ENGLISH | PHYSICAL },
  { courses: MATHEMATICS | CHEMICAL },
  { courses: ENGLISH | PHYSICAL | CHEMICAL }
]

const findStudentsViaCourses = (courses) =>
  return students.filter((student) => student.courses & courses === courses)
}

findStudentsViaCourses(ENGLISH | PHYSICAL)
// [
//   { courses: ENGLISH | PHYSICAL },
//   { courses: ENGLISH | PHYSICAL | CHEMICAL }
// ]
```

首先，我們要先把各個課程的值都改成 2 的次方倍（第 1 - 4 行）
然後將原本的陣列改成位元運算的交集（第 7 - 11 行）
最後將篩選的方法改成位元運算的連集（第 15 行）
程式整個變得超簡潔對吧！

但，怎麼做到的呢？

```javascript
const ENGLISH = 0b0001
const MATHEMATICS = 0b0010
const PHYSICAL = 0b0100
const CHEMICAL = 0b1000
```

我們可以發現，每個課程剛好只有一個位元是 `1`，換句話說就是一個位元代表一個課程，我們只要看哪個位元是 `1` 就可以知道是哪個課程了。

那如果是多個課程呢？

```javascript
const ENGLISH_AND_MATHEMATICS = ENGLISH | MATHEMATICS // 0b0011
```

我們剛剛已經知道一個位元就代表一個課程，那如東多個位元是 `1`，就代表是多個課程啦～

篩選又是怎麼做到的呢？

```javascript
ENGLISH_AND_MATHEMATICS & ENGLISH // 0b0011 & 0b0001 = 0b0001
ENGLISH_AND_MATHEMATICS & CHEMICAL // 0b0011 & 0b1000 = 0b0000
```

看了上面的例子應該就可以很發現，我們同樣利用了一個位元代表一個課程的原理，去比較兩者是否擁有相同的課程來做到交集的判斷。

## 結語

位元運算不只可以用在程式碼裡面，其實也可以用在資料庫查詢唷！
試想看看剛剛的 `students` 如果存在資料庫裡，又要做正規劃的話，那資料表會是什麼樣子的呢？
利用位元運算的話，竟然只要一張表就可以完成了耶！很神奇對吧～
大家也試著開始把位元運算運用在自己的專案裡吧！
