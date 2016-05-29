Title: 16_Markdown语法例子
Date: 2016-05-29 09:31:37
Author: peter
Postid: 16
Slug: Markdown
Nicename: Markdown

# Markdown语法例子



# h1
## h2
### h3

---

**B**

*I*

~~S~~

++U++

==Mark==

---
hr

> Quote
> Quote

- UL
- UL
- UL

1. Ol
2. OL
3. OL

- [ ] Incomplete task list
- [x] complete task list

[link](http://note.youdao.com/)
![image](http://note.youdao.com/favicon.ico)


```
code
code
code
```

Table
header 1 | header 2 | header 3
---|---|---
row 1 col 1 | row 1 col 2 | row 1 col 3
row 2 col 1 | row 2 col 2 | row 2 col 3

math
```math
E = mc^2
```



---

### 图表Markdown
####graph
```
graph LR
A-->B
A-->B
B-->c
```

#### sequence Diagram
```
sequenceDiagram
A->>B: How are you?
B->>A: Great!
```

#### gantt
```
gantt
dateFormat YYYY-MM-DD
section S1
T1: 2014-01-01, 9d
section S2
T2: 2014-01-11, 9d
section S3
T3: 2014-01-02, 9d
```

---