---
layout: post
title: A Fresh Approach to Deal with Browser's Caching
date: 2015-05-30 11:12:00.000000000 +07:00
categories: dev
tags: web
---
เว็บบราวเซอร์สมัยใหม่ถูกสร้างมาให้พยายาม **แคช** ข้อมูลที่เป็น static เช่น CSS, Javascript หรือแม้แต่รูปภาพ (มักไม่ cache ไฟล์ html) เพื่อให้การโหลดครั้งต่อไปทำได้เร็วขึ้นแน่นอนว่าก็เป็นเรื่องดี​ แต่ก็ย่อมมีข้อเสียตามมาคือ เมื่อเราแก้ไขเว็บ (แก้ทั้ง html, js, css) แล้วอัพโหลดผลการแก้ไขไปยังเซิฟเวอร์อาจจะทำให้ผู้ใช้เข้าไปใช้เว็บไม่ได้ หรือเข้าได้แต่ผลการแสดงไม่ถูกต้อง เนื่องก็ด้วย html ได้อันใหม่แต่ว่า js กับ css อาจจะยังได้ของที่อยู่ใน cache อยู่ ซึ่งแท้จริงแล้วผู้ใช้สามารถแก้ปัญหานี้ได้ด้วยการ refresh หน้าจอเพียงเท่านั้น browser ก็จะขอข้อมูลสดใหม่จาก server ซึ่งก็จะแก้ปัญหานี้ได้ แต่การแสดงหน้าเว็บที่ **ไม่พร้อมใช้งาน** ก็เป็นสิ่งที่รับไม่ได้เช่นกัน

เท่าที่ทราบเราไม่สามารถ **บังคับ** ให้ browser **รีเฟรช** ข้อมูลที่ cache จากคำสั่งทาง server โดยตรงได้ เพราะว่าสิทธิ์การแคชเป็นของ browser (และการ cache เป็นเรื่องดี เราก็ไม่อยากปิดระบบนี้) เช่น การโหลดไฟล์ สมมติ app.js ซึ่งได้ถูก cache ไว้ก่อนแล้ว ถ้า cache ยังไม่หมดอายุ browser จะไม่ถามไปยัง server ด้วยซ้ำว่าอันนี้เป็นของล่าสุดหรือเปล่า ? เพราะฉะนั้นในเมื่อ browser ไม่ถาม *เราก็หมดสิทธิ์สั่งการ*

แต่เดี๋ยวก่อน... เพราะว่ามีสิ่งหนึ่งที่ browser มักไม่ cache คือไฟล์ html นั่นเอง และการไม่ถูก cache ของ html นี่เองที่นำไปสู่การแก้ปัญหานี้! การแก้ปัญหาคือการใส่ query string ต่อท้าย resource ที่เราต้องการโหลดเช่น แทนที่จะเรียกแค่ app.js เราก็ต่อท้าย query string ไปด้วยเช่น `app.js?version=10` ซึ่งแท้จริงแล้วการเปลี่ยน query string ก็ไม่ได้ทำให้เราได้ไฟล์ที่แตกต่างกันแต่อย่างใด เพราะยังไงมันก็ไปเรียก app.js มาให้เราอยู่แล้ว (เพราะว่าเราไม่ได้เขียนความหมายของ query string ไว้ มันก็ถูก ignore ไป) แต่ความต่างอยู่ที่ browser เพราะว่า ***browser ถือว่า query string ต่างกันจะใช้ cache ร่วมกันไม่ได้*** เช่น `app.js?version=10` กับ `app.js?version=11` ก็จะไม่ถือว่าเป็นไฟล์เดียวกันเสียด้วยซ้ำ ด้วยเหตุนี้ หากเราต้องการ refresh cache ใน browser เราก็แค่เปลี่ยน query string ของไฟล์นั้น browser ก็จำต้องเรียก request มายัง server เพื่อขอไฟล์นี้อีกรอบ

## ตัวอย่างแรก
แต่ก่อนเราจะเขียนแบบนี้

```
<head>
  <script src="app.js"></script>
</head>
```

เราก็แก้เป็น

```
<head>
  <script src="app.js?v=10"></script>
</head>
```

การใส่ `v=10` หรือว่า `version=10` หรืออะไรก็แล้วแต่ไม่ได้มีคุณค่าเชิงคุณภาพต่อ browser หรือ server แต่อย่างใด เพราะว่าฝั่ง server ก็ ignore query string นี้เหมือนกัน

เย่... เหมือนว่าเราแก้ปัญหาได้แล้ว ก็แค่ใส่ `?v=x` ไปท้ายทุก ๆ ไฟล์ที่เราเรียก เช่น ไฟล์ `.css` `.js` เราก็จะหมดปัญหานี้ แล้วพอเราแก้ไฟล์พวกนี้เราก็แค่มา**ไล่แก้ทุก ๆ ที่**ให้เป็น `?v=y` แทน ... ฟังดูเริ่มเหนื่อย 

เพราะว่าที่จริงมันก็ควรจะง่ายขนาดที่มันแก้ให้เราเองเลย หรือไม่ก็แก้ด้วยมือให้น้อยที่ ๆ สุด มีหลายคนเสนอวิธีแก้แบบ fully-auto แต่ว่าต้องใช้ php เข้าช่วย ซึ่งในเคสผมผมไม่ต้องการให้มี php เข้ามายุ่งเลยแม้แต่น้อยทุกอย่างต้องถูกกำหนดก่อนมีการ deployment ลง server (ผมต้องการให้ front-end เป็น static ทั้งหมด การมี php เข้ามาเป็นสิ่งที่รับไม่ได้ ๕๕๕) ผมจึงไม่เลือกใช้วิธีดังกล่าว

## แบบที่น่าจะดีกว่า
แต่เนื่องจากผมใช้ [gulp](http://gulpjs.com/) อยู่แล้ว จึงได้ไอเดียว่า แทนที่จะใส่

```
<head>
  <script src="app.js?v=10"></script>
</head>
```

เปลี่ยนเป็นเป็น placeholder บางแบบ แล้วค่อยให้ gulp มา string replace ให้เป็นเลขที่กำหนดดีกว่ามั้ย เพราะจะได้แก้เลขแค่ที่เดียวแต่ว่าแก้ค่านี้ได้ทั้ง project เช่น

```
<head>
  <script src="app.js?v=<#__VERSION__#>"></script>
</head>
```

ถ้าเป็นแบบนี้เราก็แค่โหลด plugin สำหรับ [replace string](https://github.com/lazd/gulp-replace) ของ gulp แล้วตั้งค่าให้มัน สนใจเฉพาะไฟล์ที่เป็น html หรือไม่ก็ js เพราะใน project ผมมีแค่จาก html หรือ js ที่จะเรียกโหลดไฟล์อื่น ๆ ต่อได้ (บางกรณีอาจจะมี css ที่สามารถโหลดอันอื่นได้เช่นกัน แต่ไม่ใช่กรณีนี้)

ซึ่งผมก็จะแก้ gulpfile.js ของตัวเองเล็กน้อยโดยเพิ่ม ให้มีการ replace ก่อนที่จะมีการแปลงโค้ด jade เป็น html 
**หมายเหตุ:** ผมใช้ jade แทนการเขียน html

```
var replace = require('gulp-replace');
var VERSION = '10';

gulp.task('jade', function () {
  var dest = rootdir + 'html/';
  return gulp.src(rootdir + 'jade/**/*.jade')
    ... may be smoething here ...
    // replace any '<#_VERSION_#>' to '10' (according to VERSION)
    .pipe(replace('<#_VERSION_#>', VERSION))
    // compile jade
    .pipe(jade())
    .pipe(gulp.dest(dest))
    ... may be smoething here ...
});

```

วิธีนี้ *ไม่ใช่* วิธีที่ดีที่สุด เพราะว่า

1. มันยังไม่ automatic 100% เรายังต้องแก้ค่าตัวแปร `VERSION` อยู่ เมื่อต้องการเอาชนะ cache ของ browser
2. หากมีการแก้ไฟล์เพียงไฟล์เดียว แต่เราก็ต้องแก้ค่า `VERSION` แล้ว ก็จะส่งผมให้ cache ของทั้งเว็บหายไป คือต้องโหลดใหม่ทั้งเว็บ ซึ่งแท้จริงแล้วก็ควรจะโหลดใหม่เฉพาะไฟล์ ๆ นั้นไฟล์เดียวก็พอแล้ว เราอาจจะเขียน gulpfile ให้ดีกว่านี้ก็อาจจะแก้ปัญหานี้ได้ แต่ไม่ได้แสดงให้ดูในที่นี้
