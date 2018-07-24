---
Author: public domain
Publisher: 
Copyright: 
Email: 
Version: 2.1
Encoding: ascii
License: cc0
PoshCode ID: 5415
Published Date: 2016-09-09t22
Archived Date: 2016-05-19t09
---

# saturated arithmetic - 

## Description

saturated addition, subtraction, multiplication, and casting from numeric types to integer types using clamping to handle overflows.

## Comments



## Usage



## TODO



## class

``

## Code

`#
 #
 <#
 Version 2.1:
 - fixes a couple of typos
 
 After loading, use it like (where [T] is a primitive integer type):
 
 [sat[T]]::From($a)
 [sat[T]]::Add($a,$b)
 [sat[T]]::Sub($a,$b)
 [sat[T]]::Mul($a,$b)
 
 #>
 
 Add-Type -ReferencedAssemblies System.Numerics -TypeDefinition @'
 using System;
 using System.Numerics;
 
 public static class sat<T> where T : struct, IComparable<T>, IEquatable<T>, IConvertible {
 	internal static readonly Func<T,T,T> add_;
 	internal static readonly Func<T,T,T> sub_;
 	internal static readonly Func<T,T,T> mul_;
 	internal static readonly Func<sbyte,T> i8_;
 	internal static readonly Func<byte,T> u8_;
 	internal static readonly Func<short,T> i16_;
 	internal static readonly Func<ushort,T> u16_;
 	internal static readonly Func<int,T> i32_;
 	internal static readonly Func<uint,T> u32_;
 	internal static readonly Func<long,T> i64_;
 	internal static readonly Func<ulong,T> u64_;
 	internal static readonly Func<float,T> f32_;
 	internal static readonly Func<double,T> f64_;
 	internal static readonly Func<decimal,T> m_;
 	internal static readonly Func<BigInteger,T> b_;
 
 	public static T Add(T a, T b) { return add_(a, b); }
 	public static T Sub(T a, T b) { return sub_(a, b); }
 	public static T Mul(T a, T b) { return mul_(a, b); }
 
 	public static T From(sbyte value) { return i8_(value); }
 	public static T From(byte value) { return u8_(value); }
 	public static T From(short value) { return i16_(value); }
 	public static T From(ushort value) { return u16_(value); }
 	public static T From(int value) { return i32_(value); }
 	public static T From(uint value) { return u32_(value); }
 	public static T From(long value) { return i64_(value); }
 	public static T From(ulong value) { return u64_(value); }
 	public static T From(float value) { return f32_(value); }
 	public static T From(double value) { return f64_(value); }
 	public static T From(decimal value) { return m_(value); }
 	public static T From(BigInteger value) { return b_(value); }
 
 	static sat() {unchecked{
 		add_ = (Func<sbyte,sbyte,sbyte>)((sbyte a, sbyte b) => {
 			int ret = a + b;
 			if (ret > sbyte.MaxValue) ret = sbyte.MaxValue;
 			if (ret < sbyte.MinValue) ret = sbyte.MinValue;
 			return (sbyte)ret;
 		}) as Func<T,T,T>;
 		if (add_ != null) {
 			sub_ = (Func<sbyte,sbyte,sbyte>)((sbyte a, sbyte b) => {
 				int ret = a - b;
 				if (ret > sbyte.MaxValue) ret = sbyte.MaxValue;
 				if (ret < sbyte.MinValue) ret = sbyte.MinValue;
 				return (sbyte)ret;
 			}) as Func<T,T,T>;
 			mul_ = (Func<sbyte,sbyte,sbyte>)((sbyte a, sbyte b) => {
 				int ret = a * b;
 				if (ret > sbyte.MaxValue) ret = sbyte.MaxValue;
 				if (ret < sbyte.MinValue) ret = sbyte.MinValue;
 				return (sbyte)ret;
 			}) as Func<T,T,T>;
 			i8_ = (Func<sbyte,sbyte>)((sbyte value) => { return value; }) as Func<sbyte,T>;
 			u8_ = (Func<byte,sbyte>)((byte value) => { return (sbyte)value >= 0 ? (sbyte)value : sbyte.MaxValue; }) as Func<byte,T>;
 			i16_ = (Func<short,sbyte>)((short value) => { return value <= sbyte.MinValue ? sbyte.MinValue : value <= sbyte.MaxValue ? (sbyte)value : sbyte.MaxValue; }) as Func<short,T>;
 			u16_ = (Func<ushort,sbyte>)((ushort value) => { return value <= sbyte.MaxValue ? (sbyte)value : sbyte.MaxValue; }) as Func<ushort,T>;
 			i32_ = (Func<int,sbyte>)((int value) => { return value <= sbyte.MinValue ? sbyte.MinValue : value <= sbyte.MaxValue ? (sbyte)value : sbyte.MaxValue; }) as Func<int,T>;
 			u32_ = (Func<uint,sbyte>)((uint value) => { return value <= (byte)sbyte.MaxValue ? (sbyte)value : sbyte.MaxValue; }) as Func<uint,T>;
 			i64_ = (Func<long,sbyte>)((long value) => { return value <= sbyte.MinValue ? sbyte.MinValue : value <= sbyte.MaxValue ? (sbyte)value : sbyte.MaxValue; }) as Func<long,T>;
 			u64_ = (Func<ulong,sbyte>)((ulong value) => { return value <= (byte)sbyte.MaxValue ? (sbyte)value : sbyte.MaxValue; }) as Func<ulong,T>;
 			f32_ = (Func<float,sbyte>)((float value) => { return value <= sbyte.MinValue ? sbyte.MinValue : value >= sbyte.MaxValue ? sbyte.MaxValue : checked((sbyte)value); }) as Func<float,T>;
 			f64_ = (Func<double,sbyte>)((double value) => { return value <= sbyte.MinValue ? sbyte.MinValue : value >= sbyte.MaxValue ? sbyte.MaxValue : checked((sbyte)value); }) as Func<double,T>;
 			m_ = (Func<decimal,sbyte>)((decimal value) => { return value <= sbyte.MinValue ? sbyte.MinValue : value >= sbyte.MaxValue ? sbyte.MaxValue : (sbyte)value; }) as Func<decimal,T>;
 			b_ = (Func<BigInteger,sbyte>)((BigInteger value) => { return value <= (long)sbyte.MinValue ? sbyte.MinValue : value >= (long)sbyte.MaxValue ? sbyte.MaxValue : (sbyte)value; }) as Func<BigInteger,T>;
 			return;
 		}
 
 		add_ = (Func<byte,byte,byte>)((byte a, byte b) => {
 			int ret = a + b;
 			if (ret > byte.MaxValue) ret = byte.MaxValue;
 			return (byte)ret;
 		}) as Func<T,T,T>;
 		if (add_ != null) {
 			sub_ = (Func<byte,byte,byte>)((byte a, byte b) => {
 				int ret = a - b;
 				if (ret < 0) ret = 0;
 				return (byte)ret;
 			}) as Func<T,T,T>;
 			mul_ = (Func<byte,byte,byte>)((byte a, byte b) => {
 				int ret = a * b;
 				if (ret > byte.MaxValue) ret = byte.MaxValue;
 				return (byte)ret;
 			}) as Func<T,T,T>;
 			i8_ = (Func<sbyte,byte>)((sbyte value) => { return value >= 0 ? (byte)value : (byte)0; }) as Func<sbyte,T>;
 			u8_ = (Func<byte,byte>)((byte value) => { return value; }) as Func<byte,T>;
 			i16_ = (Func<short,byte>)((short value) => { return value <= 0 ? (byte)0 : value <= byte.MaxValue ? (byte)value : byte.MaxValue; }) as Func<short,T>;
 			u16_ = (Func<ushort,byte>)((ushort value) => { return value <= byte.MaxValue ? (byte)value : byte.MaxValue; }) as Func<ushort,T>;
 			i32_ = (Func<int,byte>)((int value) => { return value <= 0 ? (byte)0 : value <= byte.MaxValue ? (byte)value : byte.MaxValue; }) as Func<int,T>;
 			u32_ = (Func<uint,byte>)((uint value) => { return value <= byte.MaxValue ? (byte)value : byte.MaxValue; }) as Func<uint,T>;
 			i64_ = (Func<long,byte>)((long value) => { return value <= 0 ? (byte)0 : value <= byte.MaxValue ? (byte)value : byte.MaxValue; }) as Func<long,T>;
 			u64_ = (Func<ulong,byte>)((ulong value) => { return value <= byte.MaxValue ? (byte)value : byte.MaxValue; }) as Func<ulong,T>;
 			f32_ = (Func<float,byte>)((float value) => { return value <= 0 ? (byte)0 : value >= byte.MaxValue ? byte.MaxValue : checked((byte)value); }) as Func<float,T>;
 			f64_ = (Func<double,byte>)((double value) => { return value <= 0 ? (byte)0 : value >= byte.MaxValue ? byte.MaxValue : checked((byte)value); }) as Func<double,T>;
 			m_ = (Func<decimal,byte>)((decimal value) => { return value <= 0 ? (byte)0 : value >= byte.MaxValue ? byte.MaxValue : (byte)value; }) as Func<decimal,T>;
 			b_ = (Func<BigInteger,byte>)((BigInteger value) => { return value.Sign <= 0 ? (byte)0 : value >= (long)byte.MaxValue ? byte.MaxValue : (byte)value; }) as Func<BigInteger,T>;
 			return;
 		}
 
 		add_ = (Func<short,short,short>)((short a, short b) => {
 			int ret = a + b;
 			if (ret > short.MaxValue) ret = short.MaxValue;
 			if (ret < short.MinValue) ret = short.MinValue;
 			return (short)ret;
 		}) as Func<T,T,T>;
 		if (add_ != null) {
 			sub_ = (Func<short,short,short>)((short a, short b) => {
 				int ret = a - b;
 				if (ret > short.MaxValue) ret = short.MaxValue;
 				if (ret < short.MinValue) ret = short.MinValue;
 				return (short)ret;
 			}) as Func<T,T,T>;
 			mul_ = (Func<short,short,short>)((short a, short b) => {
 				int ret = a * b;
 				if (ret > short.MaxValue) ret = short.MaxValue;
 				if (ret < short.MinValue) ret = short.MinValue;
 				return (short)ret;
 			}) as Func<T,T,T>;
 			i8_ = (Func<sbyte,short>)((sbyte value) => { return value; }) as Func<sbyte,T>;
 			u8_ = (Func<byte,short>)((byte value) => { return value; }) as Func<byte,T>;
 			i16_ = (Func<short,short>)((short value) => { return value; }) as Func<short,T>;
 			u16_ = (Func<ushort,short>)((ushort value) => { return value >= 0 ? (short)value : short.MaxValue; }) as Func<ushort,T>;
 			i32_ = (Func<int,short>)((int value) => { return value <= short.MinValue ? short.MinValue : value <= short.MaxValue ? (short)value : short.MaxValue; }) as Func<int,T>;
 			u32_ = (Func<uint,short>)((uint value) => { return value <= (ushort)short.MaxValue ? (short)value : short.MaxValue; }) as Func<uint,T>;
 			i64_ = (Func<long,short>)((long value) => { return value <= short.MinValue ? short.MinValue : value <= short.MaxValue ? (short)value : short.MaxValue; }) as Func<long,T>;
 			u64_ = (Func<ulong,short>)((ulong value) => { return value <= (ushort)short.MaxValue ? (short)value : short.MaxValue; }) as Func<ulong,T>;
 			f32_ = (Func<float,short>)((float value) => { return value <= short.MinValue ? short.MinValue : value >= short.MaxValue ? short.MaxValue : checked((short)value); }) as Func<float,T>;
 			f64_ = (Func<double,short>)((double value) => { return value <= short.MinValue ? short.MinValue : value >= short.MaxValue ? short.MaxValue : checked((short)value); }) as Func<double,T>;
 			m_ = (Func<decimal,short>)((decimal value) => { return value <= short.MinValue ? short.MinValue : value >= short.MaxValue ? short.MaxValue : (short)value; }) as Func<decimal,T>;
 			b_ = (Func<BigInteger,short>)((BigInteger value) => { return value <= (long)short.MinValue ? short.MinValue : value >= (long)short.MaxValue ? short.MaxValue : (short)value; }) as Func<BigInteger,T>;
 			return;
 		}
 
 		add_ = (Func<ushort,ushort,ushort>)((ushort a, ushort b) => {
 			int ret = a + b;
 			if (ret > ushort.MaxValue) ret = ushort.MaxValue;
 			return (ushort)ret;
 		}) as Func<T,T,T>;
 		if (add_ != null) {
 			sub_ = (Func<ushort,ushort,ushort>)((ushort a, ushort b) => {
 				int ret = a - b;
 				if (ret < 0) ret = 0;
 				return (ushort)ret;
 			}) as Func<T,T,T>;
 			mul_ = (Func<ushort,ushort,ushort>)((ushort a, ushort b) => {
 				uint ret = (uint)a * (uint)b;
 				if (ret > ushort.MaxValue) ret = ushort.MaxValue;
 				return (ushort)ret;
 			}) as Func<T,T,T>;
 			i8_ = (Func<sbyte,ushort>)((sbyte value) => { return value >= 0 ? (ushort)value : (ushort)0; }) as Func<sbyte,T>;
 			u8_ = (Func<byte,ushort>)((byte value) => { return value; }) as Func<byte,T>;
 			i16_ = (Func<short,ushort>)((short value) => { return value >= 0 ? (ushort)value : (ushort)0; }) as Func<short,T>;
 			u16_ = (Func<ushort,ushort>)((ushort value) => { return value; }) as Func<ushort,T>;
 			i32_ = (Func<int,ushort>)((int value) => { return value <= 0 ? (ushort)0 : value <= ushort.MaxValue ? (ushort)value : ushort.MaxValue; }) as Func<int,T>;
 			u32_ = (Func<uint,ushort>)((uint value) => { return value <= ushort.MaxValue ? (ushort)value : ushort.MaxValue; }) as Func<uint,T>;
 			i64_ = (Func<long,ushort>)((long value) => { return value <= 0 ? (ushort)0 : value <= ushort.MaxValue ? (ushort)value : ushort.MaxValue; }) as Func<long,T>;
 			u64_ = (Func<ulong,ushort>)((ulong value) => { return value <= ushort.MaxValue ? (ushort)value : ushort.MaxValue; }) as Func<ulong,T>;
 			f32_ = (Func<float,ushort>)((float value) => { return value <= 0 ? (ushort)0 : value >= ushort.MaxValue ? ushort.MaxValue : checked((ushort)value); }) as Func<float,T>;
 			f64_ = (Func<double,ushort>)((double value) => { return value <= 0 ? (ushort)0 : value >= ushort.MaxValue ? ushort.MaxValue : checked((ushort)value); }) as Func<double,T>;
 			m_ = (Func<decimal,ushort>)((decimal value) => { return value <= 0 ? (ushort)0 : value >= ushort.MaxValue ? ushort.MaxValue : (ushort)value; }) as Func<decimal,T>;
 			b_ = (Func<BigInteger,ushort>)((BigInteger value) => { return value.Sign <= 0 ? (ushort)0 : value >= (long)ushort.MaxValue ? ushort.MaxValue : (ushort)value; }) as Func<BigInteger,T>;
 			return;
 		}
 
 		add_ = (Func<int,int,int>)((int a, int b) => {
 			long ret = (long)a + (long)b;
 			if (ret > int.MaxValue) ret = int.MaxValue;
 			if (ret < int.MinValue) ret = int.MinValue;
 			return (int)ret;
 		}) as Func<T,T,T>;
 		if (add_ != null) {
 			sub_ = (Func<int,int,int>)((int a, int b) => {
 				long ret = (long)a - (long)b;
 				if (ret > int.MaxValue) ret = int.MaxValue;
 				if (ret < int.MinValue) ret = int.MinValue;
 				return (int)ret;
 			}) as Func<T,T,T>;
 			mul_ = (Func<int,int,int>)((int a, int b) => {
 				long ret = (long)a * (long)b;
 				if (ret > int.MaxValue) ret = int.MaxValue;
 				if (ret < int.MinValue) ret = int.MinValue;
 				return (int)ret;
 			}) as Func<T,T,T>;
 			i8_ = (Func<sbyte,int>)((sbyte value) => { return value; }) as Func<sbyte,T>;
 			u8_ = (Func<byte,int>)((byte value) => { return value; }) as Func<byte,T>;
 			i16_ = (Func<short,int>)((short value) => { return value; }) as Func<short,T>;
 			u16_ = (Func<ushort,int>)((ushort value) => { return value; }) as Func<ushort,T>;
 			i32_ = (Func<int,int>)((int value) => { return value; }) as Func<int,T>;
 			u32_ = (Func<uint,int>)((uint value) => { return (int)value >= 0 ? (int)value : int.MaxValue; }) as Func<uint,T>;
 			i64_ = (Func<long,int>)((long value) => { return value <= int.MinValue ? int.MinValue : value <= int.MaxValue ? (int)value : int.MaxValue; }) as Func<long,T>;
 			u64_ = (Func<ulong,int>)((ulong value) => { return value <= (uint)int.MaxValue ? (int)value : int.MaxValue; }) as Func<ulong,T>;
 			f32_ = (Func<float,int>)((float value) => { return value <= int.MinValue ? int.MinValue : value >= -(float)int.MinValue ? int.MaxValue : checked((int)value); }) as Func<float,T>;
 			f64_ = (Func<double,int>)((double value) => { return value <= int.MinValue ? int.MinValue : value >= int.MaxValue ? int.MaxValue : checked((int)value); }) as Func<double,T>;
 			m_ = (Func<decimal,int>)((decimal value) => { return value <= int.MinValue ? int.MinValue : value >= int.MaxValue ? int.MaxValue : (int)value; }) as Func<decimal,T>;
 			b_ = (Func<BigInteger,int>)((BigInteger value) => { return value <= (long)int.MinValue ? int.MinValue : value >= (long)int.MaxValue ? int.MaxValue : (int)value; }) as Func<BigInteger,T>;
 			return;
 		}
 
 		add_ = (Func<uint,uint,uint>)((uint a, uint b) => {
 			ulong ret = (ulong)a + (ulong)b;
 			if (ret > uint.MaxValue) ret = uint.MaxValue;
 			return (uint)ret;
 		}) as Func<T,T,T>;
 		if (add_ != null) {
 			sub_ = (Func<uint,uint,uint>)((uint a, uint b) => {
 				long ret = (long)a - (long)b;
 				if (ret < 0) ret = 0;
 				return (uint)ret;
 			}) as Func<T,T,T>;
 			mul_ = (Func<uint,uint,uint>)((uint a, uint b) => {
 				ulong ret = (ulong)a * (ulong)b;
 				if (ret > uint.MaxValue) ret = uint.MaxValue;
 				return (uint)ret;
 			}) as Func<T,T,T>;
 			i8_ = (Func<sbyte,uint>)((sbyte value) => { return value >= 0 ? (byte)value : 0U; }) as Func<sbyte,T>;
 			u8_ = (Func<byte,uint>)((byte value) => { return value; }) as Func<byte,T>;
 			i16_ = (Func<short,uint>)((short value) => { return value >= 0 ? (ushort)value : 0U; }) as Func<short,T>;
 			u16_ = (Func<ushort,uint>)((ushort value) => { return value; }) as Func<ushort,T>;
 			i32_ = (Func<int,uint>)((int value) => { return value >= 0 ? (uint)value : 0U; }) as Func<int,T>;
 			u32_ = (Func<uint,uint>)((uint value) => { return value; }) as Func<uint,T>;
 			i64_ = (Func<long,uint>)((long value) => { return value <= 0 ? 0U : value <= uint.MaxValue ? (uint)value : uint.MaxValue; }) as Func<long,T>;
 			u64_ = (Func<ulong,uint>)((ulong value) => { return value <= uint.MaxValue ? (uint)value : uint.MaxValue; }) as Func<ulong,T>;
 			f32_ = (Func<float,uint>)((float value) => { return value <= 0 ? 0U : value >= 4294967296.0f ? uint.MaxValue : checked((uint)value); }) as Func<float,T>;
 			f64_ = (Func<double,uint>)((double value) => { return value <= 0 ? 0U : value >= uint.MaxValue ? uint.MaxValue : checked((uint)value); }) as Func<double,T>;
 			m_ = (Func<decimal,uint>)((decimal value) => { return value <= 0 ? 0U : value >= uint.MaxValue ? uint.MaxValue : (uint)value; }) as Func<decimal,T>;
 			b_ = (Func<BigInteger,uint>)((BigInteger value) => { return value.Sign <= 0 ? 0U : value >= (long)uint.MaxValue ? uint.MaxValue : (uint)value; }) as Func<BigInteger,T>;
 			return;
 		}
 
 		add_ = (Func<long,long,long>)((long a, long b) => {
 			long ret = a + b;
 			long ov = (ret < 0) ? long.MaxValue : long.MinValue;
 			if (((a ^ ret) & (b ^ ret)) < 0) ret = ov;
 			return ret;
 		}) as Func<T,T,T>;
 		if (add_ != null) {
 			sub_ = (Func<long,long,long>)((long a, long b) => {
 				long ret = a - b;
 				long ov = (ret < 0) ? long.MaxValue : long.MinValue;
 				if (((a ^ b) & (a ^ ret)) < 0) ret = ov;
 				return ret;
 			}) as Func<T,T,T>;
 			mul_ = (Func<long,long,long>)((long a, long b) => {
 				if (a > 0) {
 					if (b >= 0) return b <= long.MaxValue / a ? a*b : long.MaxValue;
 					return b >= long.MinValue / a ? a*b : long.MinValue;
 				} else if (a < 0) {
 					if (b <= 0) return b >= long.MaxValue / a ? a*b : long.MaxValue;
 					return a >= long.MinValue / b ? a*b : long.MinValue;
 				}
 				return 0;
 			}) as Func<T,T,T>;
 			i8_ = (Func<sbyte,long>)((sbyte value) => { return value; }) as Func<sbyte,T>;
 			u8_ = (Func<byte,long>)((byte value) => { return value; }) as Func<byte,T>;
 			i16_ = (Func<short,long>)((short value) => { return value; }) as Func<short,T>;
 			u16_ = (Func<ushort,long>)((ushort value) => { return value; }) as Func<ushort,T>;
 			i32_ = (Func<int,long>)((int value) => { return value; }) as Func<int,T>;
 			u32_ = (Func<uint,long>)((uint value) => { return value; }) as Func<uint,T>;
 			i64_ = (Func<long,long>)((long value) => { return value; }) as Func<long,T>;
 			u64_ = (Func<ulong,long>)((ulong value) => { return (long)value >= 0 ? (long)value : long.MaxValue; }) as Func<ulong,T>;
 			f32_ = (Func<float,long>)((float value) => { return value <= long.MinValue ? long.MinValue : value >= -(float)long.MinValue ? long.MaxValue : checked((long)value); }) as Func<float,T>;
 			f64_ = (Func<double,long>)((double value) => { return value <= long.MinValue ? long.MinValue : value >= -(double)long.MinValue ? long.MaxValue : checked((long)value); }) as Func<double,T>;
 			m_ = (Func<decimal,long>)((decimal value) => { return value >= long.MaxValue ? long.MaxValue : value <= long.MinValue ? long.MinValue : (long)value; }) as Func<decimal,T>;
 			b_ = (Func<BigInteger,long>)((BigInteger value) => { return value >= long.MaxValue ? long.MaxValue : value <= long.MinValue ? long.MinValue : (long)value; }) as Func<BigInteger,T>;
 			return;
 		}
 
 		add_ = (Func<ulong,ulong,ulong>)((ulong a, ulong b) => {
 			return a + b >= a ? a + b : ulong.MaxValue;
 		}) as Func<T,T,T>;
 		if (add_ != null) {
 			sub_ = (Func<ulong,ulong,ulong>)((ulong a, ulong b) => {
 				return a - b <= a ? a - b : 0UL;
 			}) as Func<T,T,T>;
 			mul_ = (Func<ulong,ulong,ulong>)((ulong a, ulong b) => {
 				if (b == 0) return 0UL;
 				return a <= ulong.MaxValue / b ? a*b : ulong.MaxValue;
 			}) as Func<T,T,T>;
 			i8_ = (Func<sbyte,ulong>)((sbyte value) => { return value >= 0 ? (byte)value : 0UL; }) as Func<sbyte,T>;
 			u8_ = (Func<byte,ulong>)((byte value) => { return value; }) as Func<byte,T>;
 			i16_ = (Func<short,ulong>)((short value) => { return value >= 0 ? (ushort)value : 0UL; }) as Func<short,T>;
 			u16_ = (Func<ushort,ulong>)((ushort value) => { return value; }) as Func<ushort,T>;
 			i32_ = (Func<int,ulong>)((int value) => { return value >= 0 ? (uint)value : 0UL; }) as Func<int,T>;
 			u32_ = (Func<uint,ulong>)((uint value) => { return value; }) as Func<uint,T>;
 			i64_ = (Func<long,ulong>)((long value) => { return value >= 0 ? (ulong)value : 0UL; }) as Func<long,T>;
 			u64_ = (Func<ulong,ulong>)((ulong value) => { return value; }) as Func<ulong,T>;
 			f32_ = (Func<float,ulong>)((float value) => { return value <= 0 ? 0UL : value >= 18446744073709551616.0f ? ulong.MaxValue : checked((ulong)value); }) as Func<float,T>;
 			f64_ = (Func<double,ulong>)((double value) => { return value <= 0 ? 0UL : value >= 18446744073709551616.0 ? ulong.MaxValue : checked((ulong)value); }) as Func<double,T>;
 			m_ = (Func<decimal,ulong>)((decimal value) => { return value <= 0 ? 0UL : value >= ulong.MaxValue ? ulong.MaxValue : (ulong)value; }) as Func<decimal,T>;
 			b_ = (Func<BigInteger,ulong>)((BigInteger value) => { return value.Sign <= 0 ? 0UL : value >= ulong.MaxValue ? ulong.MaxValue : (ulong)value; }) as Func<BigInteger,T>;
 			return;
 		}
 
 		throw new SystemException();
 	}}
 }
 '@
`

