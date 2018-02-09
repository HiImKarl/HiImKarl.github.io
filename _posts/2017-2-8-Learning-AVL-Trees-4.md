---
layout: single
title: "AVL Trees -- 4/4: Addition Features and Benchmarking"
---

# Reverse Iterator

Our reverse iterator is identical to the forwards iterator, except that the increment and decrement operators are the reverse of the ones of the forwards iterator. The ```Begin``` function is also slightly different, here is what it looks like.

```cpp
template <typename T>
ReverseIterator<T> RBegin(Node<T> *root)
{
	ReverseIterator<T> it {root};
	move_it_rightmost(&it);
	return it;
}
```

# Fancy typedefs

It's useful to provide some easily accessible type names scoped from the ```Node``` class.

```cpp
template <typename T>
struct Node {
	typedef T                 value_type;
	typedef T&              reference;
	typedef T&&               rv_reference;
	typedef T const&          const_reference;
	typedef Iterator<T>         iterator;
	typedef Iterator<T const>       const_iterator;
	typedef ReverseIterator<T>      reverse_iterator;
	typedef ReverseIterator<T const>  const_reverse_iterator;
	// stuff...
};
```

# Benchmarks

We will benchmark four methods:
1. Find
2. Insertion
3. Deletion
4. Iteration

There are many tools/frameworks you could use to benchmark, we will simply time the code with the standard library ```high_precision_clock```. We can compare our AVL tree to the standard library ```set``` container, which is typically (but not always) implemented as a Red-Black tree. I am testing this on Ubuntu 16.04 with g++ 6.3, and no compiler optimization flags.

If you don't know what a red black tree is, it supports the same methods with the same time complexity as an AVL tree, but usually AVL trees are faster at finding nodes and slower at inserting/deleting them. That is roughly how we should expect the performances to compare. That being said, it is highly probably certain optimizations/tradeoffs were made to ```std::set```, so our expectations will probably be wrong.

If you want to run the benchmarking code, cd into the root directory (/AVL) of the sample code and run:

```
make benchmark
```

## Find

Here is the results of the ```Find```. This is roughly what we expected if we were comparing AVL trees to Red Black trees. Our code runs about 3-3.5 times faster.

```
------------------------------- Find ------------------------------
  Number of Nodes   --          AVL           --          STL            
       100000       --        0.0124s         --        0.0497s 
      1000000       --        0.1525s         --        0.5591s 
     10000000       --        2.1993s         --        6.8327s 
```


## Insertion

```
----------------------------- Insertion ---------------------------
  Number of Nodes   --          AVL           --          STL            
       100000       --        0.0488s         --        0.0845s 
      1000000       --        0.6065s         --        0.9933s 
     10000000       --        7.6535s         --        11.719s 
```

So this is somewhat unexpected, in that our method is actually faster. Is is definitely possible that ```std::set``` adds additional overhead to optimize for iterations, as that is for most purposes the most frequent use case of the container. I suspect our code will run faster for deletion as well.

## Deletion

Yep, now the question is just __how much__ faster ```std::set``` will be at iteration.

```
----------------------------- Deletion ----------------------------
  Number of Nodes   --          AVL           --          STL            
       100000       --        0.0348s         --        0.0833s 
      1000000       --        0.3994s         --        0.9957s 
     10000000       --        4.8205s         --        11.396s 
```

## Iteration

Uhhhhhhhhhhhhhhhh nothing to see here.

 ``
----------------------------- Iteration ---------------------------
  Number of Nodes   --          AVL           --          STL            
       100000       --        0.0036s         --        3.36e-07s
      1000000       --        0.0358s         --        3.25e-07s
     10000000       --        0.3752s         --        3.42e-07s
```
...

Wait, the STL times aren't changing. Is the compiler optimizing something away? When I change the benchmark, to the code below, the results don't change much, and I don't **think** the compiler would optimize this loop away. In fact, the results of STL iteration is so fast, the times are probably smaller than the uncertainity in ```high_resolution_clock```.
```cpp
	// in STL benchmark func
	volatile ulong sum = 0;
	auto start = chrono::high_resolution_clock::now();
	for (;it != stl_set.end(); ++it) { sum += 1; }
	auto end = chrono::high_resolution_clock::now();

```

So what happens when if I time nothing?
```cpp
	auto start = chrono::high_resolution_clock::now();
	auto end = chrono::high_resolution_clock::now();
	cout <<  chrono::duration<double>(end - start).count() << "s\n";	
```

Result: 3.95e-07s

Hmm, I'm not sure what to make of this. It could be that ```std::set``` is even __faster__ than e-7, but I don't know. I'm going to have to look at the disassembly, but not right now, beacuse I'll have to go learn x86 first.

# Conclusion

That's basically it for AVL trees. If you want to challenge yourself, see if you can implement the potential optimizations or modify the design of the tree to make iteration faster. If you think this was actually interesting, leave a comment, and I'll continue implementing random data structures or algorithms. Thanks for reading.

* [AVL 1](https://hiimkarl.github.io//Learning-AVL-Trees-1/)
* [AVL 2](https://hiimkarl.github.io//Learning-AVL-Trees-2/)
* [AVL 3](https://hiimkarl.github.io//Learning-AVL-Trees-3/)
* [AVL 3](https://hiimkarl.github.io//Learning-AVL-Trees-4/)
