---
layout: post

title: 查找算法

date: 2016-09-16 15:32:24.000000000 +09:00

---

二分法查找算法 时间复杂度为 lgN 理由是每次将N分成一半，需要X次，最坏情况最后剩下1个。反过来。就是将一个每次放大一倍，经过X次 等于 N . 公式：2^x = N -> O = lgN .