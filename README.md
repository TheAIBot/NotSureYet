# Count true bools in an array

Although it's a simple problem to solve there's multiple ways to do it and and the performance of the different methds varies a lot. Here i will look through some options i went through when i had to solve the problem.

### Branching code

I believe the most obvious solution to the problem is to use an if statement and increment if the indexed bool is true. It's simple and everyone understands how it works, but it's also the slowest of all the solutions i've found so far. Comming in at over zzzX times slower than the fastest method so far.

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

### Branchless code

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

### Intrinsics

### Vectorized

#### AVX-512
