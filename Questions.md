Here is an example for the Bellman-Ford algorithm and the counting to infinity problem, say we have this network (links A-B, B-C, C-D, D-E all have cost 1, link from A-E has cost 100)
```
  1v    2v    3v    4v
A --- B --- C --- D --- E
 \_____________________/
         ^100
```
In this state that cost100 link won't be used, it is much cheaper to send data through B,C,D.

# Part 1 -  Bellman-Ford algorithm in well behaved network
## 1A) perform algorithm by hand
Perform 3 iterations of the Bellman-Ford algorithm for routers A,B,C,D trying to reach router E. Each iteration you should have a table that looks something like this:
```
#  A.      B.      C.      D
 (B,??), (C,??), (D,??), (E,??)
 (E,??), (A,??), (B,??), (C,??)
```
Where `#` is the iteration number, the `??` Will be the costs to get a message to E by sending it to the specified next hop. For instance the first iteration D can reach E directly through 1 hop so there should be a `(E, 4)` under the D column.

## 1B) draw spanning tree
By looking at the network we can envision that the link from A to E won't get used so the direction graph for each router to send data to E will look something like this once the algorithm has converged:
```
  1v    2v    3v    4v
A ->- B ->- C ->- D ->- E
 \_______x_____________/
         ^100
```
However it takes more than 3 iterations to reach this state.
1. what does the directed graph look like after each iteration of the algorithm you did in part 1A?
2. If you continued the algorithm another step would you expect it to produce the optimal graph shown above? Why? Does this mean the algorithm would be completely stable after 4 iterations?


### 1C) - recognize that Bellman-Ford is kind of bad at its only job
After 3 iterations of the Bellman-Ford algorithm in question 1 we ended up with this data for router D:
```
  D
(E, 4)
(C,10)
```
The (E,4) is clearly the direct route which is the one that is used.
1. what does the (C,10) represent? What path starts from D, next travels to C and ends up at E with a total cost of 10? 
2. If the link from D to E broke then D would start sending traffic for E through C by using that forwarding entry, is this problematic? Would you expect the traffic to reach E in that case?


# Question 2 - What if a link breaks?
Say we've been running the Bellman-Ford algorithm until it converged to this state:
```
0  A.      B.      C.      D
 (B, 10),(C, 9), (D, 7),  XXXX <- we are about to break this connection
 (E,100),(A,11), (B,11), (C,10)
```
Now say we broke the link between D and E so the network looks like this:
```
  1v    2v    3v     v broken
A --- B --- C --- D     E
 \_____________________/
         ^100
```
## 2A) Continue the bellman-ford algorithm after the break
Using the given initial state, continue using the algorithm for 3 or 4 steps.

The first step is that D's minimum cost changed so it will update its neighbour C about that.

## 2B) How long would it take to converge again?
We know that after this breakage that the only way for data to reach E is through the expensive link from A, and after performing 3 to 4 steps we know it didn't converge that quickly.  

1. What condition is required to make the algorithm converge in this new scenario? 
2. Give a rough estimate for how many iterations it might take for the algorithm to stabilize again or at least meet the condition mentioned above.
3. If we increased the weight of the A-E link by 3 orders of magnitude (from 100 to 100000) would you expect it to take 3 orders of magnitude more iterations to converge or some other increase factor?

### 2C) Is the Bellman-Ford algorithm kind of trash?
1. How might you describe the conditions under which the Bellman-Ford algorithm works well?
2. What would have been different if instead of breaking the link completely we increased the cost to around 100? 
3. We know that when starting from scratch it takes at most N-1 iterations where N is the number of routers in the network to converge, should we just scrap everything and start from scratch if any network link goes down?


# 3 Is there an alternative?
(This is not going to cover anything on our exam so you can skip this unless you are actually interested in fixing the issue)

Imagine a system that works very similarly to the Bellman-Ford algorithm with the following alteration:

1. When A advertises its forwarding table costs to B, it also includes a boolean indicating whether B is the next hop or not
2. When B receives this forwarding info from A, if B was previously sending to A and A is now indicating it is sending to B, then B will set the cost to send through A to infinity.
3. If a router doesn't know how to reach a host at all it sends infinity as its cost (this isn't new but possibly important to make explicit)

Does this set of rules fix the problem?

Consider this network with a loop:
```
A - B - D
 \ /
  C 
```
Legend for routing table:
- `*` means updated since last time
- `$` means current best
- `&` means updated and current best

All links are weight 1 at a stable state the connection table is as follows:
```
0  A.     B.     C
 $B,2$, (A,3), $B,2$
 (C,3), (C,3), (A,3)
        $D,1$
```
If B-D breaks then it changes to this
```
1  A.     B.     C
 $B,2$, $A,3$, $B,2$
 (C,3), (C,3), (A,3)
```
Using the modified Bellman-Ford algorithm does this count to infinity?
