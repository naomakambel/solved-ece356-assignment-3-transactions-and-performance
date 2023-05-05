Download Link: https://assignmentchef.com/product/solved-ece356-assignment-3-transactions-and-performance
<br>
Consider a small variation of the university database schema we have been using through the course. It is a small database for a university course scheduling system, providing details about courses, offerings, instructors, classrooms and departments.

Instructor (instID, instName, deptID, sessional)

Course (courseID, courseName, deptID)

Prereq (courseID, prereqID)

Offering (courseID, section, termCode, roomID, instID, enrollment)

Classroom (roomID, building, room, capacity) Department (deptID, deptName, faculty) Explanation:

<ul>

 <li>Instructor defines a unique instructor ID, his/her name, department and sessional status.</li>

 <li>Course defines a unique course ID, the course name, the department offering that offers the course.</li>

 <li>Prereq defines any prerequisites.</li>

 <li>Offering defines an actual offering of a course; the offering comprises the courseID being offered, the section number (integers starting at 1), the term code (the standard UW 4-digit term code: the first three digits define the year (add 1900 to get the year), and the fourth digit is the month in which the course starts (1 (Jan), 5 (May), or 9 (Sept) for Winter, Spring, and Fall offerings, respectively), the room where the section meets, the instructor, and the number of students enrolled.</li>

 <li>Classroom defines a unique room ID, together with the building, room number and room capacity.</li>

 <li>Department identifies a unique department ID and its name.</li>

</ul>

SQL code for this database, together with primary and foreign keys, has been provided in the source file createUni.sql.

In such a database it would be reasonable that we would want to collect multiple SQL statements together and form a single transaction. For example, there are two sections of ECE 356, but the enrollment between them is quite unbalanced. It would therefore be reasonable to want to move some of the enrollment from one section to another. Doing so, however, should be contingent on there being sufficient room within the classroom receiving the additional students. We would therefore want something like the following:

BEGIN;

update Offering set Enrollment = Enrollment – 20 where courseID=”ECE356″ and section=2 and termCode=1191;

update Offering set Enrollment = Enrollment + 20 where courseID=”ECE356″ and section=2 and termCode=1191; COMMIT;

though obviously we would need some additional work to ensure that the capacity had not been exceeded.

In order to do transactions using the command-line interface (CLI) the first thing that is required is to turn off autocommit since by default the CLI treats each operation as a transaction. To find the value of any system variable, use the “show variables” command:

mysql&gt; show variables like “autocommit”; show variables like “autocommit”;

+—————+——-


| Variable_name | Value |

+—————+——-


| autocommit                   | ON            |

+—————+——-+ 1 row in set (0.00 sec)

Since we do not want it on, we turn it off:

mysql&gt; set autocommit=0; set autocommit=0;

Query OK, 0 rows affected (0.00 sec)

mysql&gt; show variables like “autocommit”; show variables like “autocommit”;

+—————+——-


| Variable_name | Value |

+—————+——-


| autocommit                    | OFF          |

+—————+——-+ 1 row in set (0.01 sec)

See https://dev.mysql.com/doc/refman/8.0/en/server-system-variables.html for further details on system variables. This change enables the use of transactions with the CLI (or when executing from a .sql source file.

Part1: We wish to examine the effect of different isolation levels. The default isolation level is REPEATABLE-READ. To change it, use the command: set [global|session] transaction_isolation [level]

where the isolation level is one of:

READ-UNCOMMITTED

READ-COMMITTED

REPEATABLE-READ

SERIALIZABLE

Now, make two separate connections to the same database. In one, we will change the enrollment for one of the courses within a transaction:

BEGIN; update Offering set Enrollment = Enrollment – 20 where courseID=”ECE356″ and section=2 and termCode=1191;

Now, before doing the second update, and before doing the commit, in the second session connected to the database, determine what happens when you attempt to see what the enrollment is for the course. In particular, you should determine what the different results are for the different possible combinations of isolation level and report these results in your Assignment 3 submission.

Part 2: For the second part of this assignment, you are required to create a stored procedure that does the following:

<ul>

 <li>Accepts five input parameters: courseID, section1, section2, termCode, quantity</li>

 <li>execute as a transaction, the following:</li>

 <li>if (courseID,section1,termCode) or (courseID,section2,termCode) do not exist in Offering, or quantity is 0 or less or section1 = section2, set the error code to -1.</li>

 <li>attempt to reduce the enrollment in section1 by “quantity”; if the result is a negative enrollment, set the error code to -2.</li>

 <li>attempt to increase the enrollment in section2 by “quantity”; if the result is that enrollment in section2 exceeds room capacity, then set the error code to -3.</li>

 <li>if there are any errors, rollback the transaction</li>

 <li>if there are no errors, set the error code to 0 and commit the transaction.</li>

</ul>

You should call your procedure “switchSection” and you should develop a set of test cases to verify its correct functionality.

Can you achieve the same effect as the stored procedure, but by using checks and/or triggers? If so, submit the SQL code for this. If not, explain, briefly, why not.

You should submit your code and test cases as part of your Assignment 3 submission.

Part 3 The third part of this assignment requires you to become familiar with the performance schema part of MySQL. In particular, we want to know what the actual timed measurements are for queries with and without indexes. For this, we will use the Lahman Baseball database and focus on the queries from Assignment 2 as well as one other:

select count(playerID) from Master where birthYear is null or birthYear = “” or birthMonth is null or birthMonth = “” or birthDay is null or birthDay = “”;

select playerID,sum(salary) as totalPay from Salaries left outer join Appearances using (playerID,yearID,teamID) left outer join Managers using (playerID,yearID,teamID) where G is null and

G_all is null

group by playerID order by totalPay desc limit 3;

select nameFirst,nameLast,max(RBI) from Batting

inner join Master using (playerID) where HR = 0 limit 1;

To determine database performance, you need to study the “MySQL Performance Schema” chapter in the Reference Manual. (It is chapter 22 in v5.6, 25 in v5.7, and 26 in v8.0). To help get a quick start, the following will be useful:

<ul>

 <li>performance schema.setup objects contains the list of things being measured. If you add your events in your database to this, MySQL will start to measure the performance of transactions in your database:</li>

</ul>

insert into setup_objects values (’EVENT’,’lahman2016’,’%’,’YES’,’YES’);

(In my case I added “lahman2016” as that is the name of the database I want information about.)

<ul>

 <li>Timing information can be acquired from event transactions history where the time a transaction took to execute is timer end – timer start; this is measure in picoseconds, but is only actually accurate to microseconds, so you should divide the result by 1,000,000 to get it in microseconds.</li>

 <li>First, determine the performance without any indexes. Then consider various possible indexes, including those implied by primary and foreign keys. When measuring the performance, you should do at least five measurements, average them, and also report the minimum and maximum.</li>

</ul>

You should submit a report of what your performance measurements are as the third part of your Assignment 3 submission.

Part 4: Using your chess database from Assignment 2, answer the following queries:

<ol>

 <li>What fraction of games does white win?</li>

 <li>What is average number of moves per game when white wins? And if black wins?</li>

 <li>What fraction of games start with a pawn move?</li>

 <li>How many moves, on average, does white make before moving one of his/her Knights? Bishops?</li>

</ol>

Rooks (Castles)?

<ol start="5">

 <li>And black?</li>

</ol>

You may wish to reconsider how your stored “moved” in your database in order to complete this part of the assignment.