# Count true bools in an array

Although it's a simple problem to solve there's multiple ways to do it and and the performance of the different methds varies a lot. Here i will look through some options i went through when i had to solve the problem.

|          Method |     N |          Mean |       Error |      StdDev | Ratio | RatioSD |
|---------------- |------ |--------------:|------------:|------------:|------:|--------:|
|       **ScalarSum** | **10000** | **35,047.290 ns** | **243.8858 ns** | **216.1984 ns** |  **1.00** |    **0.00** |
| ScalarUnsafeSum | 10000 |  5,536.032 ns |  43.8261 ns |  36.5968 ns |  0.16 |    0.00 |
|       VectorSum | 10000 |    428.468 ns |   5.6014 ns |   5.2396 ns |  0.01 |    0.00 |

## Branching code

I believe the most obvious solution to the problem is to use an if statement and increment if the indexed bool is true. It's simple and everyone understands how it works, but it's also the slowest of all the solutions i've found so far. Comming in at over 81X times slower than the fastest method i've found.

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

##### Why so slow?

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



## Vectorized

##### AVX-512

