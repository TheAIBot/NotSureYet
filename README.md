# Count true bools in an array

Although it's a simple problem to solve there's multiple ways to do it and and the performance of the different methds varies a lot. Here i will look through some options i went through when i had to solve the problem.

|          Method |        N |        Mean |     Error |      StdDev |      Median | Ratio | RatioSD |
|---------------- |--------- |------------:|----------:|------------:|------------:|------:|--------:|
|       ScalarSum | 10000000 | 40,881.0 us | 892.99 us | 2,223.87 us | 39,835.3 us | 81.42 |    3.25 |
|   ScalarFastSum | 10000000 |  5,494.6 us |  30.61 us |    28.63 us |  5,501.4 us |  9.98 |    0.11 |
| ScalarPopCntSum | 10000000 |    909.5 us |   5.50 us |     4.87 us |    910.5 us |  1.65 |    0.02 |
|       VectorSum | 10000000 |    550.8 us |   6.60 us |     6.17 us |    550.1 us |  1.00 |    0.00 |


## Branching code

I believe the most obvious solution to the problem is to use an if statement and increment if the indexed bool is true. It's simple and everyone understands how it works, but it's also the slowest of all the solutions i've found so far. Comming in at over 81X times slower than the fastest method described here.

```
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

Insert benchmark where the method is tested with random, all true and all false arrays.

Something about how many cycles it uses per element in the array.

#### Why so slow?

Introduce intel vtune?
The random test is much slower than the other two because the branch predictor guesses wrong  50% of the time. It's the branch preictors job to find patterns in code execution and use that to predict what instructions will be executed, but there is no pattern in the code execution as it relies on the values in the array which are random.


## Branchless code

Branching is slow. We can take advantage of the fact that C# is very strict about the vlues of a bool. Unless you change it yourself, then true has the value `1` and false is `0`. If we convert the bool array to a byte array, then we can sum up all the bytes and get the sum of all true bools that way.

```
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

```
public int CountTruesByAdding(bool[] bools)
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

```
public int CountTruesByAdding(bool[] bools)
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

