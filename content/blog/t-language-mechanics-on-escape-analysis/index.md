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

## Escape Mechanics

## Readability

## Compiler Reporting

## Conclusion


### Refferences
([Language Mechanics On Escape Analysis](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html))