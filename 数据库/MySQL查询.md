# MySQL查询

https://www.cnblogs.com/runnermark/p/9258140.html

## 表

![NhHswq.md.png](https://s1.ax1x.com/2020/06/29/NhHswq.md.png)

```sql
# 年级
create table class_grade(
    gid int primary key auto_increment,
    gname varchar(16) not null unique
);

# 班级信息
create table class(
    cid int primary key auto_increment,
    caption varchar(16) not null,
    grade_id int not null,
    foreign key(grade_id) references class_grade(gid)
);

# 学生信息
create table student(
    sid int primary key auto_increment,
    sname varchar(16) not null,
    gender enum('女','男') not null default '女',
    class_id int not null,
    foreign key(class_id) references class(cid)
);

# 教师信息
create table teacher(
    tid int primary key auto_increment,
    tname varchar(16) not null
);

# 课程信息
create table course(
    cid int primary key auto_increment,
    cname varchar(16) not null,
    teacher_id int not null,
    foreign key(teacher_id) references teacher(tid)
);

# 成绩
create table score(
    sid int not null unique auto_increment,
    student_id int not null,
    course_id int not null,
    score int not null,
    primary key(student_id,course_id),
    foreign key(student_id) references student(sid)
    on delete cascade
    on update cascade,
    foreign key(course_id) references course(cid)
    on delete cascade
    on update cascade
);

# 教师任课班级
create table teach2cls(
    tcid int not null unique auto_increment,
    tid int not null,
    cid int not null,
    primary key(tid,cid),
    foreign key(tid) references teacher(tid)
    on delete cascade
    on update cascade,
    foreign key(cid) references class(cid)
    on delete cascade
    on update cascade
);
```





## 查询

#### 1、查询学生总人数

```sql
select count(sid) from student;
```

#### 2、查询“生物”课程和“物理”课程成绩都及格的学生id和姓名

```sql
select
  sid,
  sname
from
  student
where
  sid in(
    select
      score.student_id
    from
      score
    inner join course on score.course_id=course.cid
    where
      course.cname in(
        '生物',
        '物理'
        )
        and score.score >= 60
        group by
            score.student_id
        having
            count(course_id)=2
      );
```

#### 3、查询每个年级的班级数，取出班级数最多的前三个年级

```sql
SELECT
	gid,
	gname 
FROM
	class_grade 
WHERE
	gid IN ( 
		SELECT grade_id 
		FROM class 
		GROUP BY grade_id 
		ORDER BY count( cid ) ASC 
		) LIMIT 3;
```

#### 4、查询平均成绩最高和最低的学生的id和姓名以及平均成绩

```sql
SELECT
	student.sid,
	student.sname,
	t1.avg_score 
FROM
	student
	INNER JOIN (
		(
		SELECT
			student_id,
			avg( score ) AS avg_score 
		FROM
			score 
		GROUP BY
			student_id 
		HAVING
			avg( score ) 
		ORDER BY
			avg( score ) DESC 
			LIMIT 1 
		) 
        UNION
		(
		SELECT
			student_id,
			avg( score ) AS avg_score 
		FROM
			score 
		GROUP BY
			student_id 
		HAVING
			avg( score ) 
		ORDER BY
			avg( score ) ASC 
			LIMIT 1 
		) 
	) AS t1 ON student.sid = t1.student_id;
```

#### 5、查询每个年级的学生人数

```sql
SELECT
	gname,
	sum( class_num ) 
FROM
	class_grade
	INNER JOIN (
	SELECT
		grade_id,
		cid,
		count( sid ) AS class_num 
	FROM
		class
		INNER JOIN student ON student.class_id = class.cid 
	GROUP BY
		cid 
	) AS t1 ON t1.grade_id = class_grade.gid 
GROUP BY
	grade_id;
```

#### 6、查询每位学生的学号，姓名，选课数，平均成绩

```sql
# 这里只用左连接
select student.sid,sname,class_num,avg_score
from student
left join (
	select student_id,count(course_id) as class_num,avg(score) as avg_score
	from score
	group by student_id
) as t1 on t1.student_id=student.sid;
```

#### 7、查询学生编号为“2”的学生的姓名、该学生成绩最高的课程名、成绩最低的课程名及分数、

```SQL
SELECT
	sname,
	cname,
	score 
FROM
	student
	INNER JOIN (
	SELECT
		score.student_id,
		course_id,
		score,
		cname 
	FROM
		score
		INNER JOIN course ON course.cid = score.course_id 
	WHERE
		student_id = 2 
		AND score.score IN (
			( SELECT max( score.score ) FROM score WHERE score.student_id = 2 ),
			( SELECT min( score.score ) FROM score WHERE score.student_id = 2 ) 
		) 
	) AS t1 ON t1.student_id = student.sid
```

#### 8、查询姓“李”的老师的个数和所带班级数

```SQL
SELECT
	teacher_num,
	class_num 
FROM
	(
	SELECT
		count( t2.tid ) AS teacher_num,
		sum( t2.teacher_class_num ) AS class_num 
	FROM
		(
		SELECT
			t1.tid,
			count( cid ) AS teacher_class_num 
		FROM
			( SELECT teacher.tid FROM teacher WHERE tname LIKE '李%' ) AS t1
			INNER JOIN teach2cls ON teach2cls.tid = t1.tid 
		GROUP BY
			tid 
		) AS t2 
	) AS t3;
```

#### 9、查询班级数小于5的年级id和年级名

```sql
select gid,gname
from class_grade
where gid in(
	select grade_id
	from class
	group by grade_id
	having count(cid)<5
);
```

#### 10、查询班级信息，包括班级id、班级名称、年级、年级级别 ( 12为低年级，34为中年级，56为高年级 )

示例：

```
班级id   班级名称   年级 年级级别
    1  一年一班   一年级    低
```

```sql
select
	cid as '班级id',
	caption as '班级名称',
	gname as '年级',
	case
		when gid<=2 then '低'
		when gid<=4 then '中'
		else '高'
		end as '年级级别'
from class
inner join class_grade on class.grade_id = class_grade.gid;
```

#### 11、查询学过“张三”老师2门课以上的同学的学号、姓名

