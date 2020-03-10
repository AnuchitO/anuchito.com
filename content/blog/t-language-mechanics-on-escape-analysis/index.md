---
title: [ร่าง][แปล] Language Mechanics On Escape Analysis
date: "2020-03-10T07:45:00.000Z"
description: "[ร่าง][แปล] Language Mechanics On Escape Analysis"
---

# Prelude
 TODO 

# Introduction

# Heaps คืออะไร
นอกจาก Stack ที่เป็นพื้นที่ memory ไว้เก็บค่าต่างๆแล้ว Heap คือพื้นที่ของ memory ในเครื่องคอมพิวเตอร์
พื้นที่หน่วยความจำที่ถูกจองใน Heap ไม่สามารถคืนหน่วยความจำเหล่านั้นได้อัตโนมัติ
จึงทำให้ต้องทำให้มีค่าใช้จ่ายในการใช้ memory ในส่วนนี้
ค่าใช้จ่ายที่ว่านั้นสัมพันธ์กับสิ่งที่เรียกว่า Garbage collector (GC) ซึ่งเป็นตัวจัดการคืนค่าพื้นที่ ที่ไม่ได้ใช้แล้วใน Heap
เมื่อ GC ใช้ 25% ของ CPU ที่เหลืออยู่ในการทำงานเพื่อเคลียร์ค่าพื้นที่ที่ไม่ได้ใช้งานใน Heap
และ ณ ขณะ GC ทำงานมันอาจจะเกิดสิ่งที่เรียกว่า "stop the world" latency ในช่วงเวลาสั้น (microseconds)

GC ทำให้เราไม่ต้องกังวัลเกี่ยวกับการจัดการ heap memory ซึ่งมันเป็นสิ่งที่มีความซับซ้อนและเกิดข้อผิดพลาดได้ง่าย

ค่าที่ถูกจองบน Heap 

# Sharing Stacks

# Escape Mechanics

# Readability

# Compiler Reporting

# Conclusion