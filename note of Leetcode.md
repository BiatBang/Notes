## Graph
785: Partition a graph into 2 parts.
1557: Find vertices with no in-degrees.
684: Union-Find, make if an edge's 2 vertices belong to one cluster, there is a cycle
787: Dijkstra with steps recorded

## Tree
95: dfs to handle both the left and right child, still, we need to end the tree with a null as a leaf!
98: Validate Binary Search Tree: each node in the bst needs to be maintained in a range. left nodes should be less than the father and larger than a dynamic updated value
102: use a map to record each level and corresponding list, then pick them out //// or, why we not bfs???? when it concerns about level, we can always use bfs!
116: Populating Next Right Pointers in Each Node: one node's left's next is its right; one node's right's next is its next's left
1315: dfs with parent and grandparent
623: insert a layer of nodes into a tree, normal dfs.
654: recursively slice and compose, slice and compose
687: good tree question. if you need to store a global variable, but you really don't want it, you can use a 1-length array! passing the array works the same as global variable

## Math Game
779: Bit operation
650: copy all and past. Long string can be pasted by short n times

## Sliding Window
560: normal sliding window doesn't work because negative value is allowed, so use a preSum map to maintain cumulative sum, to calculate the result of a window
567: sliding window with fixed length
424: window without heading back

## DP
576: path to every grids with limited steps
650: copy all and past. Long string can be pasted by short n times
647: find palindromic strings
300: find increasing successive subarray
322: 0-1 knapsack
402: ?
542: grid to touch te surrounding grids. It's a good idea to not look back, one round from left-top to right-bottom, one round from right-bottom to left-top, then each loop only focus on the facing direction
120: Triangle: dp, node can't reach the right ancestor, since it will exceed the adjacent node

## Array
15: 3 sum, sort the array, iterate from head to len-2 as the first el, then find 2 els that + el = 0
16: very similar to 15
17: queue to mark the prev strings
55: greedy, always find the last step
565: dfs with boolean[] visited, still controlled in O(n)
611: Triplet with O(N^2), one as iterator, head and tail iterates the els before iterator
556: find next number. From tail to head, find the first valley, that's the bit to change. Then find the smallest el larger than then valley after the valley, then swap them. Then sort the substring after the valley(before), then the result is the result.
659: greedy works! first append the existing list rather than start a new head. maintain two map to record the frequency and the possible locations. 

## Queue
622: circular queue

## Trie
648: Trie to find words in the dictionary

## Greedy
646: Greedy to solve time slots problem, that promise success

## LinkedList
86: sometimes if we cut the linkedlist, we need to end the sublist, to avoid the loops
138: Copy List with Random Pointer: to make a deep copy, we can make a side-by-side copied list, then we can assign the random nodes. Finally, we pick out the copied list and **maintain the original list**
142: Linked List Cycle II: to detect a loop in a linkedlist, suppose the straight part has length m, loop part has length n, then whole length is m+n. a slow node chasing a fast node, will meet it at length n. so, the rest of the list will be m. Then, get another node from head, let it move along with the slow one, they'll meet at length m.
160: find the intersection, even if lengths differ. len(a) + len(b) == len(b) + len(a)
328: odd and even list, odd = odd.next.next, even = even.next.next, odd.next = evenHead

## Stack
402: use a stack to remove k digits to compose the smallest value

## BFS
433: bfs without layers, no need to empty the queue inside a loop. Sometimes iterating brutely is not a bad way, since there is no better way.
102: Binary Tree Level Order Traversal: bfs the tree to record each nodes on each layer
103: Binary Tree Zigzag Level Order Traversal: same as 102, but use list.add(index, element) to insert into head

## DFS
695: dfs to iterate the grids, mark the grid to 0 as visited, or invalid, then return the max

## Binary Search
540: binary search to find the only single one. find middle, locate the last one of the pair riding on middle, then see each half's length, if odd, then the result is in it. 

## String
93: Restore IP Addresses: Integer.valueOf(String value), throws exception if the string is a null or it's too big
696: it's tricky that "0001111", has 3 0's and 4 1's, so the possible result is Math.min(3, 4) = 3.

## SQL
175: normal join
176: if you want to get a "", keep it that way; if you want to get a null, add an outer select
178: one way to sort is find how many scores are >= current score, then the rank is that number. group by id can make every row individual
181: use join with on is slower than where
182: group and having
183: exclude, using not in
184: default join is inner join, works 
192: default join is inner join