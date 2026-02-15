---
title: "Zig 类型转换"
date: "2026-02-06"
description: "Zig 类型转换主要有两种：显式类型转换和隐式类型转换。显式类型转换需要使用内置函数，而隐式类型转换则是自动进行的。"
author: "uozewe"
tags: ["Zig", "类型", "转换"]
---

## 1 隐式类型转换

隐式类型转换由编译器自动完成，无需开发者显式指定。Zig 的隐式类型转换规则相对严格，只允许在特定情况下进行。

### 1.1 限制规则

Zig 只允许以下隐式类型转换：

- `*T` → `*const T`：将可变指针转换为常量指针
- `*T` → `*volatile T`：将普通指针转换为易失指针
- `*T` → `?*T`：将普通指针转换为可选指针
- 大内存对齐 → 小内存对齐：降低内存对齐要求
- 错误集合 → 超集：扩大错误集合范围

### 1.2 数值类型拓宽

当目标类型可以表示源类型的所有可能值时，Zig 允许自动转换。这通常发生在数值类型拓宽时。

```zig
// 无符号整数拓宽
const a: u8 = 250;
const b: u16 = a;  // u8 → u16
const c: u32 = b;  // u16 → u32
const d: u64 = c;  // u32 → u64
const e: u128 = d; // u64 → u128

// 有符号整数拓宽
const f: u8 = 250;
const g: i16 = f;  // u8 → i16

// 浮点数拓宽
const h: f16 = 12.34;
const i: f32 = h;  // f16 → f32
const j: f64 = i;  // f32 → f64
const k: f128 = j; // f64 → f128
```

### 1.3 数组与指针转换

Zig 支持以下数组与指针之间的隐式转换：

1. **数组指针 → 切片**：将指向数组的指针转换为切片类型
2. **数组指针 → 多项指针**：将指向数组的指针转换为指向数组元素的指针

## 2 显式类型转换

显式类型转换需要使用 Zig 的内置函数。这些函数提供了精确的类型控制，但需要开发者对转换结果负责。

### 2.1 整数类型转换

#### 2.1.1 `@intCast` - 安全转换

`@intCast` 在整数类型之间进行安全转换，如果值超出目标类型的范围，会触发运行时安全错误。

```zig
const a: i32 = 100;
const b: u8 = @intCast(a); // 100 在 u8 范围内（0 - 255），转换成功

const c: i32 = 300;
const d: u8 = @intCast(c); // 运行时错误：300 > u8 最大值 255
```

#### 2.1.2 `@truncate` - 截断转换

`@truncate` 将整数转换为更小的整数类型，直接截断高位，不进行范围检查。

```zig
const a: u16 = 0x1234;
const b: u8 = @truncate(a); // 截断高位，保留低8位 → 0x34
```

#### 2.1.3 `@sext` - 符号扩展

`@sext` 将整数符号扩展到目标类型，保留符号位，高位填充符号位的值。

```zig
const x: i8 = -1; // 二进制：11111111
const y: i16 = @sext(x); // 符号扩展为 11111111 11111111 → 仍为 -1
```

#### 2.1.4 `@zext` - 零扩展

`@zext` 将整数零扩展到目标类型，高位填充 0。

```zig
const a: u8 = 0xAB;
const b: u16 = @zext(a); // 零扩展，高位填充 0 → 0x00AB
```

### 2.2 位级转换

`@bitCast` 按位重新解释类型，不改变内存中的二进制表示，仅改变其类型解读方式。源类型和目标类型必须具有相同的大小（以字节为单位）。

```zig
const a: u32 = 0x12345678;
const b: [4]u8 = @bitCast(a); // 按位解释为4字节数组
```

### 2.3 枚举转换

#### 2.3.1 `@enumToInt` - 枚举转整数

将枚举值转换为整数，枚举值的底层类型必须是整数类型。

#### 2.3.2 `@intToEnum` - 整数转枚举

将整数转换为枚举值，整数必须在枚举值的有效范围内，否则会触发运行时错误。

### 2.4 指针转换

#### 2.4.1 `@ptrCast` - 指针类型转换

在不同的指针类型之间进行显式强制转换。

```zig
const a: u32 = 0x12345678;
const ptr: *u8 = @ptrCast(&a); // 将 *u32 转换为 *u8
```

#### 2.4.2 `@ptrToInt` - 指针转整数

将指针转换为整数，整数类型必须足够大以容纳指针的地址值。

```zig
const value: u32 = 42;
const ptr: *const u32 = &value;
const addr: usize = @ptrToInt(ptr); // 获取指针地址
```

#### 2.4.3 `@intToPtr` - 整数转指针

将整数转换为指针，整数必须是有效指针地址值。

```zig
const addr: usize = 0x1000;
const ptr: *u32 = @intToPtr(addr); // 将地址转换为指针
```

### 2.5 浮点数转换

#### 2.5.1 `@floatCast` - 浮点数类型转换

浮点数类型之间的安全转换，超出范围会触发运行时错误。

```zig
const a: f32 = 3.14;
const b: f64 = @floatCast(a); // f32 → f64
```

#### 2.5.2 `@floatToInt` - 浮点数转整数

将浮点数转换为整数，直接截断小数部分，不进行舍入。

```zig
const a: f32 = 3.9;
const b: i32 = @floatToInt(a); // 结果为 3
```

#### 2.5.3 `@intToFloat` - 整数转浮点数

将整数转换为浮点数，整数会被解释为浮点数的整数部分。

```zig
const a: i32 = 42;
const b: f32 = @intToFloat(a); // 结果为 42.0
```

### 2.6 对齐转换

#### 2.6.1 `@alignCast` - 指针对齐转换

将指针转换为具有更大对齐要求的指针类型，确保指针满足目标类型的对齐要求。

```zig
const a: u32 = 0x12345678;
const ptr: *align(4) u32 = &a;
const aligned: *align(8) u32 = @alignCast(ptr); // 确保指针8字节对齐
```

### 2.7 错误集转换

#### 2.7.1 `@errorCast` - 错误集转换

将一个错误集转换为另一个错误集，源错误集必须是目标错误集的子集。

```zig
const ErrorSet1 = error{ A, B, C };
const ErrorSet2 = error{ A, B };

const err1: ErrorSet1 = ErrorSet1.A;
const err2: ErrorSet2 = @errorCast(err1); // 成功转换，A 在 ErrorSet2 中

const err3: ErrorSet1 = ErrorSet1.C;
const err4: ErrorSet2 = @errorCast(err3); // 运行时错误：C 不在 ErrorSet2 中
```

## 3 其他类型转换

### 3.1 `@as` - 类型断言

`@as` 用于给表达式指定类型，主要用于编译期类型推断有歧义的情况。与显式转换函数不同，`@as` 不能用于运行时类型转换，它只是告诉编译器如何理解类型。

```zig
const a: u32 = 42;
const b: u64 = @as(u64, a); // 编译期类型断言
```
