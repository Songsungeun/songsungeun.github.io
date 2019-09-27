---
layout: post
title:  "Vuex Actions에서 Actions 호출"
date:   2019-05-29 15:10:42 +0900
comments : true
background: '/img/posts/webpack.png'
---

Vuex를 통해 작업을 하다보니,
Actions에서 Actions를 호출하는 경우가 생겼다.
Actions는 Promise를 return 하는데
페이지 > Actions > Actions를 호출함으로써
프로미스 체인과 같이 사용할 수도 있지만
async, await으로 정리하는게 깔끔하다.

``` javascript

```