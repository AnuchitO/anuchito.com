---
title: "[ร่าง][แปล] Language Mechanics On Escape Analysis"
date: "2020-03-10T07:45:00.000Z"
description: "[ร่าง][แปล] Language Mechanics On Escape Analysis"
---

## Prelude
 TODO 

## Introduction

## Heaps คืออะไร
หน่วยความจำในคอมพิวเตอร์ที่ใช้สำหรับรันโปรแกรมมีสองส่วนหลัก คือ Stack และ Heap.
Stack เป็นหน่วยความจำที่สามารถคืนพื้นที่หน่วยความจำที่ไม่ได้ใช้แล้วได้อัตโนมัติ
ส่วน Heap ไม่สามารถคืนหน่วยความจำได้ด้วยตนเอง จึงต้องอาศัยสิ่งที่เรียกว่า Garbage collector (GC)
เป็นตัวช่วยในการจัดการคืนพื้นที่หน่วยความจำไม่ได้ใช้แล้วให้กับระบบ
เมื่อ GC รันมันจะไช้ 25% ของ CPU ที่มีเหลืออยู่ในการทำงานเพื่อตรวจหาและคืนพื้นที่หน่วยความจำที่ไม่ได้ใช้แล้วใน Heap ให้กับระบบ
และ ณ ขณะ GC รันมันอาจะเกิดสิ่งที่เรียกว่า "stop the world" latency ในช่วงเวลาสั้นๆ (microseconds)

GC ทำให้เราไม่ต้องกังวัลเกี่ยวกับการจัดการ heap memory ซึ่งมันเป็นสิ่งที่มีความซับซ้อนและเกิดข้อผิดพลาดได้ง่าย
ค่าที่ถูกจองบน Heap ใน Go ก็ทำให้ GC ต้องทำงานเพื่อลบพื้นที่หน่วยความจำที่ไม่ได้ใช้แล้ว (หน่วยความจำตรงนั้นไม่ได้ถูกอ้างถึงแล้ว)

## Sharing Stacks
ใน Go, goroutine ไม่สามารถมี pointer ที่ใช้ไปยังหน่วยความจำใน stack ของ goroutine ตัวอื่น
เพราะว่า stack memory ของ goroutine สามารถถูกเขียบทับใหม่ได้เมื่อขนาดของ stack ต้องขยายหรือหดลง
ฉะนั้นถ้า runtime มี pointers ชี้ไปยัง stack ของ goroutine จะทำให้ต้องจัดการ "stop the world" latency
เพื่อทำการอัพเดทหน่วยความจำบน stack ยุ่งยากยุบยับเต็มไปหมด

นี่เป็นตัวอย่างของ stack ที่ถูกเขียบทับหลายๆครั้งเมื่อ stack ต้องขยายขนาด
ผลลัพธ์บรรทัดที่ 2 และ 6 จะว่า address ของ string ใน stack frame เปลี่ยนค่าไปสองครั้ง

https://play.golang.org/p/pxn5u4EBSI

  0 0x44ff88 HELLO
  1 0x44ff88 HELLO
  2 0x457f88 HELLO
  3 0x457f88 HELLO
  4 0x457f88 HELLO
  5 0x457f88 HELLO
  6 0x46ff88 HELLO
  7 0x46ff88 HELLO
  8 0x46ff88 HELLO
  9 0x46ff88 HELLO

## Escape Mechanics

## Readability

## Compiler Reporting

## Conclusion


### Refferences
([Language Mechanics On Escape Analysis](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html))