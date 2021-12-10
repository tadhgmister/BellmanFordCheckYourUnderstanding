

# 1 - circle network
```
  1v    2v    3v    4v
A --- B --- C --- D --- E
 \_____________________/
         ^100
```
## Q1A - Bellman-Ford algorithm from scratch

Initially no one knows how to reach anyone
```
0  A.      B.      C.      D
 (?,??), (?,??), (?,??), (?,??)
```

After one iteration D and A discover they can reach E through 1 hop
```
1  A.      B.      C.      D
 (B,??), (C,??), (D,??), (E, 4)
 (E,100),(A,??), (B,??), (C,??)
```

Now that A and D know how to reach E both B and C get notified too
They will calculate the distance to be the advertised cost + cost to reach that neighbour
So if D advertises cost of 4 and it takes C a cost of 3 to reach D it will use a cost of 7
Similarly for B->A->E
```
2  A.      B.      C.      D
 (B,??), (C,??), (D, 7), (E, 4)
 (E,100),(A,101),(B,??), (C,??)
```

Now that everyone has a way to reach E we will fill in the entire table in the same way we did before, cost to reach neighbour + advertised cost by neighbour to fill in the entire table
```
3  A.      B.      C.      D
 (B,102),(C, 9), (D, 7), (E, 4)
 (E,100),(A,101),(B,103),(C,10)
```
The pseudo code to perform this is basically the following
```
For each router X
   For each neighbour Y of that router
      cost to reach destination by making the first hop of X to Y = (lowest cost advertised by Y) + (cost to hop from X to Y)
```
If you looked up this answer before trying it try to advance this further until it stabilizes (an iteration is identical to the last) if you want to practice for the exam.

## Q1B - spanning tree
### 1B1 first few iterations
After one iteration only A and D can reach E
```
  1v    2v    3v    4v
A -X- B -X- C -X- D ->- E
 \>__________________>_/
         ^100
```
After 2 iterations B and C also know direct routes:
```
  1v    2v    3v    4v
A -<- B -X- C ->- D ->- E
 \>__________________>_/
         ^100
```
After the 3rd iteration B gets informed that it is much cheaper to send to 
```
  1v    2v    3v    4v
A -X- B ->- C ->- D ->- E
 \>__________________>_/
         ^100
```
### 1B2 - 4th iteration
After only 1 more iteration B will tell A that it has a better route and it will reach the optimal forwarding setup.  In general with N routers the longest possible route is N-1 hops (which in this case the optimal route from A to E goes through every router) so it takes at most N-1 iterations to reach the optimal routing setup.

However there will still be some non-optimal entries that will still update after the 4th iteration, for instance once A knows it can reach E with a cost of 10, B will update the path through A.

### Q1C - what does the (C,10) mean under that D column?
### Q1C1 - what does this path represent?
The best path from C to E is C->D->E with cost (3+4=7). C advertises this 7 to D and D then considers the path to C (cost 3) plus the advertised cost 7 so the total cost is 10. But that means that full path is D->C->D->E which goes back through D! 
### Q1C2 - is it a problem?
This shows that the Bellman-Ford algorithm does allow loops or backtracking in the routes which gives rise to the "counting to infinity problem" that we examine more in question 2.

# Q2 - a break in the chain
```
  1v    2v    3v     v broken
A --- B --- C --- D     E
 \_____________________/
         ^100
```
## Q2A - Bellman-Ford
Initially we know we have this state:
```
0  A.      B.      C.      D
 (B, 10),(C, 9), (D, 7), *____*
 (E,100),(A,11), (B,11), (C,10)
```
The first thing is that compared to last iteration the cost for D to reach E has increased so the entry (D, 7) under C will increase accordingly, the new value is 10+3 since D advertises the cost of 10 and it costs 3 to send to D
```
1  A.      B.      C.      D
 (B, 10),(C, 9), *D,13*,  ____
 (E,100),(A,11), (B,11), (C,10)
```
This changed the min cost of C so both its neighbours B and D will update
```
2  A.      B.      C.      D
 (B, 10),*C,13*, (D,13),  ____
 (E,100),(A,11), (B,11), *C,14*
```
This changes C-D but more importantly changed the min cost of B so C and A are affected
```
3  A.      B.      C.      D
 *B, 12*,(C,13), *D,17*,  ____
 (E,100),(A,11), *B,13*, (C,14)
```
Now C and A's min cost has increased which changes B and D costs to them
```
4  A.      B.      C.      D
 (B, 12),*C,15*, (D,17),  ____
 (E,100),*A,13*, (B,13), *C,16*
```
## 2B - what does this mean? 
### 2B1 - what has to happen to make it work again
Once the link from D-E breaks the only way to reach E is through the expensive link A-E. So the condition that lets traffic actually get sent to E is that A realizes this is the cheapest route to E. This means that the 100 cost from A to E has to be less than the cost of the rest of the connections that are just looping internally.
### 2B2 - how long might it take to converge?
It will take probably around 40 to 50 steps since each step increases costs by only about 2 to 4 and it only increases a given number every second step since it has to back-propagate. The specific number doesn't really matter the important thing is that it will take significantly longer to converge than if we had just started from scratch.
### 2B3 - what if we went up by orders of magnitude
It would definitely take 3 orders of magnitude longer since the rate it converges doesn't depend on that number at all, The speed it converges is roughly proportional to the ratio of the expensive link divided by the cheapest link.


## 2C) Is the Bellman-Ford algorithm kind of trash?
Yes
### 2C1) Conditions under which the Bellman-Ford algorithm works well
The Bellman-Ford algorithm works better when the difference in costs of links isn't very large, when a path that took a cost of 3-10 suddenly changes to cost 100 it may take a while to realize.

### 2C2) What if we increased the weight instead of breaking it
Absolutely nothing. It might converge to a slightly different solution but it'd still end up not using either link to reach E for a long time.

### 2C3) Should we just scrap everything and start from scratch if any network link goes down?
Probably not, in a typical network it is probably uncommon to have orders of magnitude difference between link costs so this kind of case where it converges really slowly to happen often. At least not compared to cases where the costs change drastically or drop.


# 3) An actual solution
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

This changed the min cost of B from 1 to 3 which updates A and C, but A is also told it is the next hop
```
2  A.     B.     C
 *B,?*, $A,3$, *B,4*
 $C,3$, (C,3), $A,3$
```
This means that now A and C are pointing at each other so they figure out that sending to each other is a bad idea, also A and C tells B that their cost went up
```
3  A.     B.     C
 (B,?), &A,4&, $B,4$
 *C,?*, *C,4*, *A,?*
```
At this point A doesn't know how to reach destination at all and tells this to B, as well C tells B that its cost increased
```
4  A.     B.     C
 (B,?), *A,?*, $B,4$
 (C,?), &C,4&, (A,?)
```
At this point B and C tell each other that they are the next hop so they both cancel out
```
5  A.     B.     C
 (B,?), (A,?), *B,?*
 (C,?), *C,?*, (A,?)
```
And now no one knows how to reach D so the network is stable.

