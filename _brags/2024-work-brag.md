---
title: '2024 Work Brag'
layout: post
---

### Goals for this year

1. Gain proficiency in the TiDB database optimizer and comprehend its functioning thoroughly
2. Brush up on Data Structures and Algorithms, don't be afraid of leetCode
3. Move to a new country

### Projects

#### TiDB Priority Queue

##### Weekly Summary

- 2024-01-07:
  - Finished the POC of the priority queue [tidb#49418](https://github.com/pingcap/tidb/pull/49418)
    - It also showed a very good improvement in the issue [tidb#49972](https://github.com/pingcap/tidb/issues/49972)
  - Updated the new design of the priority queue.
  - Added the basic heap implementation [tidb#50126](https://github.com/pingcap/tidb/pull/50126)
  - Added the interval util function for the priority queue [tidb#50035](https://github.com/pingcap/tidb/pull/50035)
- 2024-01-14:
  - Added some interval util functions [tidb#50035](https://github.com/pingcap/tidb/pull/50035)
- 2024-02-11:
  - Added the refresher for the priority queue [tidb#50845](https://github.com/pingcap/tidb/pull/50845)
  - Finished building the priority queue [tidb#51045](https://github.com/pingcap/tidb/pull/51045)
- 2024-02-18:
  - Merged the design of the priority queue [tidb#49018](https://github.com/pingcap/tidb/pull/49018)
- 2024-02-25:
  - Merged the priority queue for non-partitioned tables [tidb#51045](https://github.com/pingcap/tidb/pull/51045)
  - Used the correct time to calculate the last analyze time [tidb#51287](https://github.com/pingcap/tidb/pull/51287)
  - Finished the priority queue for partitioned tables [tidb#51152](https://github.com/pingcap/tidb/pull/51152)
- 2024-03-10:
  - Added the priority calculator [tidb#51346](https://github.com/pingcap/tidb/pull/51346)
  - Fixed the issue of using the wrong time to calculate the last analyze time [tidb#51395](https://github.com/pingcap/tidb/pull/51395)
  - Fixed the issue of using the wrong SQL to analyze static partitioned tables [tidb#51479](https://github.com/pingcap/tidb/pull/51479)
  - Refactored the priority queue job [tidb#51531](https://github.com/pingcap/tidb/pull/51531)
  - Enabled the priority queue by default [tidb#515327](https://github.com/pingcap/tidb/pull/51537)
- 2024-03-17:
  - Goals: Finish the testing of the priority queue
  - Achieved: Finished one benchmark test
- 2024-03-24:
  - Achieved: Finished some benchmark tests
- 2024-03-31:
  - Goals: Finish the testing of the priority queue
  - Achieved: Have not finished the testing yet
- 2024-04-07:
  - Goals: Finish the testing of the priority queue
  - Achieved: TB

#### TiDB

##### Weekly Summary

- 2024-01-07:
  - Tried to fix [tidb#38756](https://github.com/pingcap/tidb/pull/50020) in [50020](https://github.com/pingcap/tidb/pull/50020)
- 2024-01-14:
  - Made `SELECT DISTINCT SQRT(1) FROM t` work [tidb#50020](https://github.com/pingcap/tidb/pull/50020)
- 2024-01-21:
  - Set the correct type for `REMAINING_SECONDS` [tidb#50421](https://github.com/pingcap/tidb/pull/50421)
  - Added stack information for runtime error [tidb#50449](https://github.com/pingcap/tidb/pull/50449)
- 2024-02-11:
  - Made the used stats info thread-safe [tidb#51029](https://github.com/pingcap/tidb/pull/51029)
- 2024-02-18:
  - Skip create pseudo stats for partitions during auto-analyze process [tidb#51123](https://github.com/pingcap/tidb/pull/51123)
- 2024-02-25:
  - No Progress
- 2024-03-10:
  - No Progress
- 2024-03-17:
  - Goals: Fix a bug in the TiDB and read the join algorithm in TiDB
  - Achieved: No Progress
- 2024-03-24:
  - Achieved: Skip alway false DNF(Disjunctive Normal Form) condition in TiDB [tidb#51901](https://github.com/pingcap/tidb/pull/51901)
- 2024-03-31:
  - Goals: Fix a bug in the TiDB and read the hash join algorithm in TiDB
  - Achieved:
    - Fixed the stuck issue in the TiDB [tidb#52107](https://github.com/pingcap/tidb/pull/52107)
    - Removed the unnecessary atomic when generating the hash join plan [tidb#52060](https://github.com/pingcap/tidb/pull/52060)
    - Do not allow set `tidb_auto_analyze_ratio` to 0 [tidb#52190](https://github.com/pingcap/tidb/pull/52190)
- 2024-04-07:
  - Goals: Fix a bug in the TiDB and read the hash join algorithm in TiDB
  - Achieved: TBD

#### LeetCode

##### Weekly Summary

- 2024-01-07:
  - Finished [LeetCode#49](https://leetcode.com/problems/group-anagrams/description/) in TypeScript
- 2024-01-14:
  - Finished 349, 238
- 2024-01-21:
  - Finished 271, 167
- 2024-02-11:
  - Finished 1343, 0235, 0217 and 0097
- 2024-02-18:
  - Finished 1584, 0743, 0787 and 0332.
- 2024-02-25:
  - Finished all neetcode problems
  - Started solve those problems in Rust and Go again
- 2024-03-10:
  - Finished 0020, 0139, 0704 and 0206 in both Rust and Go
- 2024-03-17:
  - Goals: Finish one problem every day
  - Achieved: Finished 0143, 0139, 0226, 0104, 0208 and 0078
- 2024-03-24:
  - Achieved: Finished 0200, 0070, 0133 and 0746
- 2024-03-31:
  - Goals: Finish one problem every day
  - Achieved: Finished 1584, 0198, 0213 and 0005
- 2024-04-07:
  - Goals: Finish one problem every day
  - Achieved: TBD

#### Data Structures and Algorithms

##### Weekly Summary

- 2024-01-07:
  - Finished the lecture 1 and 2 of the course [MIT 6.006]
  - Finished the lecture 3, 4 and 5 of the course [MIT 6.006]
- 2024-02-18:
  - Finished the lecture 6, 7 and 8 of the course [MIT 6.006]
  - Finished my own implementation of the [Vector in Rust](https://github.com/hi-rustin/build-my-own-x/pull/87)
- 2024-02-25:
  - Finished the lecture 9, 10 and 11 of the course [MIT 6.006]
  - Added insert API for the Vector in Rust [build-my-own-x#95](https://github.com/hi-rustin/build-my-own-x/pull/95)
- 2024-03-10:
  - Finished the my-vec implementation in Rust
  - Finished the linked list implementation in Rust
- 2024-03-17:
  - Goals: Finish the Hash Table implementation in Rust and read the data structure book
  - Achieved: Read some chapters of the book but no progress on the Hash Table implementation
- 2024-03-24:
  - Achieved: No Progress :(
- 2024-03-31:
  - Goals: Finish the sorting algorithms in Rust and finish the graph lecture in the course [MIT 6.006]
  - Achieved: Finished the sorting algorithms in Rust and finished the graph lecture in the course [MIT 6.006]
- 2024-04-07:
  - Goals: Implement a hash map in Rust and finish the rest graph lectures in the course [MIT 6.006]
  - Achieved: TBD

[MIT 6.006]: https://www.youtube.com/playlist?list=PLUl4u3cNGP63EdVPNLG3ToM6LaEUuStEY
