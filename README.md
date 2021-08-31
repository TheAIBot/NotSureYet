# Count true bools in an array

In C# a true boolean has the least significant bit set and a false boolean has no bits set. To count the number of true bools in an array all you have to do is sum up all the set bits in the array. C# doesn't support arithmetic operations on booleans, instead it's possible to test the booleans value and execute different code depending on the result. In C# this test takes the form of an if statement where a number is incremented if the boolean is true.

```C#
public int CountTruesByBranching(bool[] bools)
{
	int sum = 0;
	for (int i = 0; i < bools.Length; i++)
	{
		if (bools[i])
		{
			sum++;
		}
	}

	return sum;
}
```



It's a simple solution and it works on all platforms. The problem is performance. The test makes it so there is now two branches of execution that the code can take. in order to explain why that's a problem i will explain a little bit about how a modern x86 CPU executes code.



In order to increase peformance, CPUs like to execute code before the previous code has even finished. In this example the CPU can do the test before it has even loaded the boolean from memory.

To do this, the CPU has to predict which branch it should






Although it's a simple problem to solve there's multiple ways to do it. Here i will look through some options i went through when i had to solve the problem. If you don't care about the journey the you can find the results in the table below and the best solution at the end of this.

|          Method |        N |        Mean | Ratio |
|---------------- |--------- |------------:|------:|
|       ScalarSum | 10.000.000 | 40,881.0 us | 81.42 |
|   ScalarFastSum | 10.000.000 |  5,494.6 us |  9.98 |
| ScalarPopCntSum | 10.000.000 |    909.5 us |  1.65 |
|       VectorSum | 10.000.000 |    550.8 us |  1.00 |

## Branching code

I believe the most obvious solution to the problem is to use an if statement and increment if the indexed bool is true. It's simple and everyone understands how it works, but it's also the slowest of all the solutions i've found so far. Comming in at over 81X times slower than the fastest method described here.

```C#
public int CountTruesByBranching(bool[] bools)
{
	int sum = 0;
	for (int i = 0; i < bools.Length; i++)
	{
		if (bools[i])
		{
			sum++;
		}
	}

	return sum;
}
```
#### Benchmark

This solution is the only one where its performance depends on the data.

|    Method |         N |      Mean |
|---------- |---------- |----------:|
| ScalarSum |    Random | 41.327 ms |
| ScalarSum |  All True |  6.654 ms |
| ScalarSum | All False |  5.616 ms |



Insert benchmark where the method is tested with random, all true and all false arrays.

Something about how many cycles it uses per element in the array.

#### Why so slow?

Introduce intel vtune?
The random test is much slower than the other two because the branch predictor guesses wrong  50% of the time. It's the branch preictors job to find patterns in code execution and use that to predict what instructions will be executed, but there is no pattern in the code execution as it relies on the values in the array which are random.


## Branchless code

Branching is slow. We can take advantage of the fact that C# is very strict about the vlues of a bool. Unless you change it yourself, then true has the value `1` and false is `0`. If we convert the bool array to a byte array, then we can sum up all the bytes and get the sum of all true bools that way.

```C#
public int CountTruesByAdding(bool[] bools)
{
	ReadOnlySpan<byte> bytes = MemoryMarshal.Cast<bool, byte>(bools);
	int sum = 0;
	for (int i = 0; i < bytes.Length; i++)
	{
		sum += bytes[i];
	}

	return sum;
}
```


## Intrinsics

The previous solution did

```C#
public int CountTruesWithPopCnt(bool[] bools)
{
	ReadOnlySpan<ulong> longs = MemoryMarshal.Cast<bool, ulong>(bools);
	int sum = 0;
	for (int i = 0; i < longs.Length; i++)
	{
		sum += (int)Popcnt.X64.PopCount(longs[i]);
	}

	ReadOnlySpan<byte> bytes = MemoryMarshal.Cast<bool, byte>(bools);
	for (int i = longs.Length * sizeof(ulong); i < bytes.Length; i++)
	{
		sum += bytes[i];
	}

	return sum;
}
```


## Vectorized

```C#
public int CountTruesWithAVX(bool[] bools)
{
	ReadOnlySpan<byte> bytes = MemoryMarshal.Cast<bool, byte>(bools);
	int sum = 0;
	int index = 0;

	if (Avx.IsSupported)
	{
		fixed (byte* arrayPtr = bytes)
		{
			Vector128<long> intermediateSum = Vector128<long>.Zero
			int leftOver = bools.Length % Vector256<byte>.Count;
			for (; index < bools.Length - leftOver; index += Vector256<byte>.Count)
			{
				Vector256<byte> loaded = Avx.LoadVector256(arrayPtr + index);
				Vector128<byte> addded = Sse2.Add(loaded.GetLower(), loaded.GetUpper());
				Vector128<ushort> splitSum = Sse2.SumAbsoluteDifferences(addded, Vector128<byte>.Zero);
				intermediateSum = Sse2.Add(intermediateSum, splitSum.AsInt64());
			}

			sum = (int)intermediateSum.GetElement(0) + (int)intermediateSum.GetElement(1);
		}
	}

	for (; index < bytes.Length; index++)
	{
		sum += bytes[index];
	}

	return sum;
}
```

##### AVX-512

