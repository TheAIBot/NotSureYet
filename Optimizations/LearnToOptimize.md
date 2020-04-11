# What to learn in order to optimize code

If you want to learn how to optimize code then there are a few general questions you have to learn the answer to.
This should explain what you need to learn/use in order to answer those questions.


## What's slow?
The most important part of optimization is to know what to optimize. It's a waste of effort to optimize a function that only takes 0.1% of the total execution time anyway. A profiler helps you answer this question by telling you how much the code spends in each function. Many profilers can do a lot more than just looking at the time spent. visual studio has a C# profiler that can also look at the time spent in a thread, spent waiting for a lock and how much you program allocates. The information a profiler gives you is the most important part of optimizing code.

## Why is it slow?
There can be a lot of answers to this question but in general there are three categories(the last category isn't here as i am not done with it. you can see a draft of it in the link).

* **Algorithms** Sometimes the code uses an algorithm that takes an obscene amount of time even though it isn't necessary. In order to fix this, you need to know a lot of algorithms your self. The best way here is probably to read a book and then use youtube to explain the algorithms if they are difficult to understand. I can recommend this free(well found the pdf on google) book i have read myself, [Algorithm design](http://www.cs.sjtu.edu.cn/~jiangli/teaching/CS222/files/materials/Algorithm%20Design.pdf). Books suck but you need to know a lot of algorithms in order to increase your chances of knowing an efficient one to a problem. Learning algorithms takes a lot of time and the best way to learn them is probably to take the time and read a book.

* **Data  structures and hardware utilization** It's too easy to write code that's horrible to execute for the CPU and you need to be able to know why it's horrible for the CPU. In order to know this you need to know a few things. The most important parts are:
  * You need to know what the stack and heap is. 
    * The basic gist here is that the stack is fast but small and the heap is slow but big.
  * The difference between [value types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types) and [reference types](https://docs.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types).
  * You need to know the importance of data layout.
  * Why branching (if else, switch) is slow. 
  
  Going even further you should also know about these things.
  * How fast some CPU instructions are to execute compared to others. 
    * ```float a = b * 0.5f``` is faster than ```float a = b / 2.0f``` even though they essentially do the same thing.
  * Multithreading performance pitfalls i.e. locks, false sharing.

  /*
  This subsection is not done
* **Language Internal workings** Usually a language does a lot of things behind the scenes to make programming easier. This is expecially true in C#. C# also provides a huge library of awesome classes that makes programming easier. All of this comes with its own performance pitfalls and you should know about them.
  * Linq is an amazing tool that makes programming so much faster. Unfortunatly it also makes it really easy to write slow code. The(what you thought was equivalent) code without linq can be many times faster.
  * You made a struct and placed it inside a HashSet. Hashset requires a hash from the struct. You haven't provided one so C# does it for you. It's magic and it all just works, except it's much slower than it should be. You can read why that's the case [in this stackoverflow answer](https://stackoverflow.com/a/39391290).
  
  */


## How can i make it faster?
When you know where and why the code it slow you will usually also know how to make it faster. Algorithm has a shit time complexity? replace it with another algorithm. Don't know of a better algorithm? Make your own or google your way to one. Data access is shit because the data structures are shit? replace them with better ones that makes data access more sequential or at the very least utilizes the cache better.

I don't think there is a much better answer to this. How to optimize depend a lot on the code and how it's used. If you want to be better at optimizing then you need to know more about 3 things. Algorithms, data structures and the internal working of the CPU.


### Clean code
Clean/Readable code actually go a long way with optimized code. My own experience is that optimizing code usually makes the code cleaner and simpler(not always the case ofc). In many cases optimizations remove the cruft and unessesary code that didn't need to be there anyway. Optimized code also tends to reduce the number of branches in code, or reorder them in such a way that they are easier to reason with.
If you don't know how to optimize something then try making the code simpler to begin with. The increased understand of the code after doing so can give good ideas about optimizations.

### Tests
Always have tests beforehand that you can use to verify that the result didn't change. Don't waste your own(and others) time thinking that you optimized something only to later find out that it's not working as expected. 

### Low level optimizations

* **Vectorization**
	As of writing this, C# does not auto vectorize code but it does expose the intrinsic functions so you can do it manually. A lot of things have to be true before it's worth it though. You should basically have a good grasp of **Data  structures and hardware utilization** section above, before you begin working with vectorization.
* **A better profiler**
	In some cases you will find that the visual studio C# profiler doesn't provide enough information about the performance problem. In those cases i will recommend the [intel vtune profiler](https://software.intel.com/en-us/vtune).




# Learning resources
Unfortuately i can't remember many of the resources i myself used to learn about optimization. I know you can google most of the things mentioned here and get a lot of great answers. I can recommend looking for stackoverflow answers as they are usually really easy to understand and not too long winded.

I will link to some of the resources i have used to learn which includes some focusing on C++.

* Book: [Algorithm Design](http://www.cs.sjtu.edu.cn/~jiangli/teaching/CS222/files/materials/Algorithm%20Design.pdf)
* Book: [Optimizing software in C++](https://www.agner.org/optimize/optimizing_cpp.pdf)
* Video: [CppCon 2014: Chandler Carruth "Efficiency with Algorithms, Performance with Data Structures"](https://www.youtube.com/watch?v=fHNmRkzxHWs)
* Video: [CppCon 2017: Carl Cook “When a Microsecond Is an Eternity: High Performance Trading Systems in C++”](https://www.youtube.com/watch?v=NH1Tta7purM)
* Video: [Moving Faster: Everyday Efficiency in Modern C++](https://www.youtube.com/watch?v=LFv7XwgsdLY)
