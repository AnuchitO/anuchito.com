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
เพราะว่า stack memory ของ goroutine สามารถถูกเขียบทับใหม่ได้เมื่อต้องเพิ่มหรือลดขนาดของ stack
ฉะนั้นถ้า runtime มี pointers ชี้ไปยัง stack ของ goroutine จะทำให้ต้องจัดการ "stop the world" latency
เพื่อทำการอัพเดทหน่วยความจำบน stack ยุ่งยากยุบยับมากขึ้น

นี่เป็นตัวอย่างของ stack ที่ถูกเขียบทับเมื่อต้องเพิ่มขนาดของ stack
ผลลัพธ์บรรทัดที่ 2 และ 6 จะว่า address ของ string ใน stack frame เปลี่ยนค่าไปสองครั้ง

https://play.golang.org/p/pxn5u4EBSI

```
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
```

## Escape Mechanics
ทุกครั้งที่มีการใช้ค่าที่เก็บในหน่วยความจำร่วมกันนอกเหนือขอบเขตของ stack frame ของฟังก์ชั่น,ค่านั้นจะถูกย้ายไปอยู่ในหน่วยความจำ Heap.
Escape analysis algorithms มีหน้าที่วิเคราะห์หาว่า ค่าใดควรย้ายไปอยู่ใน Heap และ
ยังคงทำให้มั่นใจได้ว่าโปรแกรมทำงานได้แม่นยำถูกต้องและมีประสิทธิภาพเช่นเดิม

ตัวอย่าง สำหรับทำความเข้าใจกลไกเบื้องหลังการทำงานของ Escap analysis.

https://play.golang.org/p/Y_VZxYteKO

##### Listing 1

```go
01 package main
02
03 type user struct {
04     name  string
05     email string
06 }
07
08 func main() {
09     u1 := createUserV1()
10     u2 := createUserV2()
11
12     println("u1", &u1, "u2", &u2)
13 }
14
15 //go:noinline
16 func createUserV1() user {
17     u := user{
18         name:  "Bill",
19         email: "bill@ardanlabs.com",
20     }
21
22     println("V1", &u)
23     return u
24 }
25
26 //go:noinline
27 func createUserV2() *user {
28     u := user{
29         name:  "Bill",
30         email: "bill@ardanlabs.com",
31     }
32
33     println("V2", &u)
34     return &u
35 }
```

คอมเมนต์ `go:noinline` เป็น directive เพื่อบอกให้คอมไพเลอร์อย่าทำการ inlining functions นี้ไปที่ main function.
เราใส่คอมเมนต์นี้เพื่อให้ตัวอย่างนี้ง่ายต่อการอธิบาย
ในบทความถัดไปจะยกตัวอย่าง side effects ของการ inlining ให้ดู

ใน Listing 1, เราจะเห็นว่าทั้งสองฟังก์ชันสร้าง user value และคืนค่ากลับไปให้คนที่เรียกมัน.
ฟังก์ชัน createUserV1() คืนค่าในรูปแบบ `value semantics`


##### Listing 2

```go
16 func createUserV1() user {
17     u := user{
18         name:  "Bill",
19         email: "bill@ardanlabs.com",
20     }
21
22     println("V1", &u)
23     return u
24 }
```

I said the function is using value semantics on the return because the user value created by this function is being copied and passed up the call stack. This means the calling function is receiving a copy of the value itself.

You can see the construction of a user value being performed on lines 17 through 20. Then on line 23, a copy of the user value is passed up the call stack and back to the caller. After the function returns, the stack looks like this.

ฟังกชัน createUserV1() คืนค่าในรูปแบบของ `value semantics`  เพราะว่าค่า user ถูกสร้างโดยฟังก์ชันนี้และถูก copied แล้วส่งกลับไปยังคนที่เรียกมัน นั่นหมายความว่า ฟังก์ชัน main จะได้รับใหม่ที่ copy ของ user ที่มาจากค่าที่ถูกสร้างเสร็จในฟังก์ชั่นนี้.

บรรทัดที่ 17 ถึง 20 คือการสร้าง user value แล้วในบรรทัดที่ 23 ทำจะทำการ copy ค่าของ user แล้วส่งกลับไปยัง call stack และกลับไปยังคนที่เรียกฟังก์ชั่นนี้
หลังจากฟังก์ชันนี้ทำงานเสร็จ stack จะมีลักษณะดังรูป

##### Figure 1
![Figure 1](https://www.ardanlabs.com/images/goinggo/81_figure1.png)

ใน Figure 1 ค่าของ user จะคงอยู่ในทั้งสอง  frames หลังจากเกิดการเรียก createUserV1
ตัวอย่างถัดไปจะเป็นการคืนค่าที่อยู่ในรูปแบบของ `pointer semantics`


## Readability

## Compiler Reporting

## Conclusion


### Refferences
([Language Mechanics On Escape Analysis](https://www.ardanlabs.com/blog/2017/05/language-mechanics-on-escape-analysis.html))