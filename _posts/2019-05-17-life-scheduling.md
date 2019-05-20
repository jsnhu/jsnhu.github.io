---
title:  "Flexible shift scheduling with Excel/JuMP: UBC LIFE Collegium"
published: false
share: false
---

*Thanks to the CAs of LIFE Collegium 2018-19 who made this project possible and for making my first year extremely enjoyable. I will thoroughly miss you all.*

[See my GitHub for the Excel sheet and full Julia code.](https://github.com/jsnhu/life-collegium-schedule)

This scheduling problem is significantly more interesting than [my first project](https://jsnhu.github.io/spl-scheduling/) because of the nature of flexible shifts. In a predetermined shift problem, the shifts are decided first and then employees provide their availability only for those shifts. However, for this flexible shift problem, employees first give their availability for all relevant working hours. Then, the shifts are created based on these working hours (and on various constraints).

### Quick Overview

### Problem Specifications

<img src="/assets/images/life-scheduling/">


Now, we use the [Taro](https://github.com/aviks/Taro.jl) package to read Excel sheets into Julia DataFrames. We read the staff, shift, and preference score tables.

### Objective Function
### Constraints
### Result
### Extensions
### Improvements
