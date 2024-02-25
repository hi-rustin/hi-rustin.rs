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

[MIT 6.006]: https://www.youtube.com/playlist?list=PLUl4u3cNGP63EdVPNLG3ToM6LaEUuStEY
