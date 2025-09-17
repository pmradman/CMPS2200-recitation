# CMPS 2200  Recitation 03

**Name (Team Member 1):**_____Petra Radmanovic________  
**Name (Team Member 2):**_____Hrishi Kabra_______



## Analyzing a recursive, parallel algorithm


You were recently hired by Netflix to work on their movie recommendation
algorithm. A key part of the algorithm works by comparing two users'
movie ratings to determine how similar the users are. For example, to
find users similar to Mary, we start by having Mary rank all her movies.
Then, for another user Joe, we look at Joe's rankings and count how
often his pairwise rankings disagree with Mary's:

|      | Beetlejuice | Batman | Jackie Brown | Mr. Mom | Multiplicity |
| ---- | ----------- | ------ | ------------ | ------- | ------------ |
| Mary | 1           | 2      | 3            | 4       | 5            |
| Joe  | 1           | **3**  | **4**        | **2**   | 5            |

Here, Joe (somehow) liked *Mr. Mom* more than *Batman* and *Jackie
Brown*, so the number of disagreements is 2:
(3 <->  2, 4 <-> 2). More formally, a
disagreement occurs for indices (i,j) when (j > i) and
(value[j] < value[i]).

When you arrived at Netflix, you were shocked (shocked!) to see that
they were using this O(n^2) algorithm to solve the problem:



``` python
def num_disagreements_slow(ranks):
    """
    Params:
      ranks...list of ints for a user's move rankings (e.g., Joe in the example above)
    Returns:
      number of pairwise disagreements
    """
    count = 0
    for i, vi in enumerate(ranks):
        for j, vj in enumerate(ranks[i:]):
            if vj < vi:
                count += 1
    return count
```

``` python 
>>> num_disagreements_slow([1,3,4,2,5])
2
```

Armed with your CMPS 2200 knowledge, you quickly threw together this
recursive algorithm that you claim is both more efficient and easier to
run on the giant parallel processing cluster Netflix has.

``` python
def num_disagreements_fast(ranks):
    # base cases
    if len(ranks) <= 1:
        return (0, ranks) #showing a vlue that is the smae (no disagreement)
    elif len(ranks) == 2:
        if ranks[1] < ranks[0]:
            return (1, [ranks[1], ranks[0]])  # found a disagreement
        else:
            return (0, ranks) #no disagreement
    # recursion
    else:
        left_disagreements, left_ranks = num_disagreements_fast(ranks[:len(ranks)//2])
        right_disagreements, right_ranks = num_disagreements_fast(ranks[len(ranks)//2:])
        #split the ranks in half until we compare 1 and 1 on either side 
        combined_disagreements, combined_ranks = combine(left_ranks, right_ranks)

        return (left_disagreements + right_disagreements + combined_disagreements,
                combined_ranks)
                #combine the rankings

def combine(left_ranks, right_ranks):
    i = j = 0
    result = []
    n_disagreements = 0
    while i < len(left_ranks) and j < len(right_ranks):
        if right_ranks[j] < left_ranks[i]: 
            n_disagreements += len(left_ranks[i:])   # found some disagreements
            result.append(right_ranks[j])
            j += 1
        else:
            result.append(left_ranks[i])
            i += 1
    
    result.extend(left_ranks[i:])
    result.extend(right_ranks[j:])
    print('combine: input=(%s, %s) returns=(%s, %s)' % 
          (left_ranks, right_ranks, n_disagreements, result))
    return n_disagreements, result

```

```python
>>> num_disagreements_fast([1,3,4,2,5])
combine: input=([4], [2, 5]) returns=(1, [2, 4, 5])
combine: input=([1, 3], [2, 4, 5]) returns=(1, [1, 2, 3, 4, 5])
(2, [1, 2, 3, 4, 5])
```

As so often happens, your boss demands theoretical proof that this will
be faster than their existing algorithm. To do so, complete the
following:

a) Describe, in your own words, what the `combine` method is doing and
what it returns.

.  
.  
.  
.  
.  The combine step merges the two sorte4d list sides togehter, and also sees any disagreements between the two list sides. It indexes through the two lists, comparing them one by one. When a disagreement is found int he right list (meaning it is less than the left ranking) we know that every consecutive left element is greater than the right elements since it is already sorted (it can count multiple disagreements at once given how many elements are already in the list). This continues on to count the disagreemnts. THe result is the total number of disagreeements and the fully sorted list. 
.  
.  
.  
.  

b) Write the work recurrence formula for `num_disagreements_fast`. Please explain how do you have this.

.  W(n) = 2 W(n/w) + O(n)

This is because at each recurrence split, the number of elements (n) gets divided in half, so b = 2. Each combine call is a linear compaorison thorugh the list leading to an overall o(n) work per run. Since we recursively call the fucniton on each split half, work outside of the recusion ends up being the pslitting and merging, which for both is overall O(n)
.  
.  
.  
.  
.  

c) Solve this recurrence using any method you like. Please explain how do you have this.

.  
.  
.  The solution ends up being W(n) = O(nlogn)

At the root level, the work is n. Every consecitiv e level, the work is split in two, but done for both sides of the tree, so 2 * n/2 or n. Given that the work ends up being n at each level, it will be equal to the total depth of the tree, or log_b n. n * log_b n which asymptotically  = O(n logn)
.  
.  
.  
.  
.  
.  
.  


d) Assuming that your recursive calls to `num_disagreements_fast` are
done in parallel, write the span recurrence for your algorithm. Please explain how do you have this.

.  
.  
.  Span simply looks at the maximum depth of one recurssive branch when run in parallel. In this case the Span equation would be S(n) = S(n/2) + O(log n) since the only differenc is we are not looking at both sides, meaning not multiplying by 2. 
.  
.  
.  
.  
.  

e) Solve this recurrence using any method you like. Please explain how do you have this.

.  
.  
.  
.  
.  When this gets solved, n + n/2 + n/4 etc = O(n). since we are looking asymptotically, and neither the outside work or the leaf work is larger, the work simply becomes S(n) = O(n) wghen ignoring constants. since O(n) + O(log n ) = n + log n, we sikmply say S(n) = O (n)
.  
.  
.  
.  
.  
.  

f) If `ranks` is a list of size n, Netflix says it will give you
lg(n) processors to run your algorithm in parallel. What is the
upper bound on the runtime of this parallel implementation? (Hint: assume a Greedy
Scheduler). Please explain how do you have this.

If we are thinking int erms of greedy processors, we can apply Brents Theorem which states that for p processors, Tp <= O(W/P + S). Since our work was = (n log n), this menas that the Tp = (n log n/log n ) + n or n + n. SInce we are thinking of this asymptotically, adding n to n would only add a constatnt, which we ignore. spo, the upper bound runtime would be O(n)