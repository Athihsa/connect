# connect

Sailors Database: (sailor:table -----> M:N)

create table sailor(sid int primary key, sname varchar(20), rating int, age int);
create table boat(bid int primary key, bname varchar(20), color varchar(20));
create table reserves( sid int, bid int, rdate date, foreign key(sid) references sailor(sid), foreign key(bid) references boat(bid));

insert into sailor values(1, 'Albert', 4, 34),(2,'Antony',7,56),(3,'Max',9,47),(4,'Alex', 3, 40),(5,'John', 10,78),(6,'Abigel', 8, 65),(7,'Berel',9,50),(8,'Ken', 5, 25);
insert into boat values(101,'Blue Miracle','Blue'),(102,'Winter Storm','White'),(103,'Crusier', 'Red'),(104,'Titanic','Black'),(105,'About Time', 'Green');
insert into reserves values(1,101,"2022-08-23"),(1,102,"2023-01-05"),(1,103,"2022-12-12"),(1,104,"2022-04-24"),(1,105,"2022-11-11"),(2,105,"2022-11-11"),(3,105,"2021-09-20"),(4,105,"2023-04-09"),(5,105,"2022-06-20"),(6,105,"2023-06-26"),(7,105,"2022-04-09"),(8,103,"2022-07-22");

Queries:
1. Find the colours of boats reserved by Albert:
         select color from boat b, sailor s, reserves r where s.sid=r.sid and b.bid=r.bid and s.sname='Albert';

2.       Find all sailor id’s of sailors who have a rating of at least 8 or reserved boat 103.
           (select s.sid from sailor s where s.rating>=8) UNION ( select sid from reserves r where bid=103);

3.       Find the names of sailors who have not reserved a boat whose name contains the string “storm”. Order the names in ascending order.
           select s.sname from sailor s where s.sid not in (select r.sid from reserves r where r.bid in(select b.bid from boat b where b.bname like "%St
orm%")) Order by s.sname;

4.       Find the names of sailors who have reserved all boats.
           select s.sname from sailor s where not exists( select * from boat b where not exists( select * from reserves r where s.sid=r.sid and b.bid=r.
bid));

5.       Find the name and age of the oldest sailor.
             select s.sname, s.age from sailor s where s.age in (select max(s.age) from sailor s);

6.       For each boat which was reserved by at least 5 sailors with age >= 40, find the boat id and the average age of such sailors.
            select b.bid, avg(s.age) from sailor s, boat b, reserves r where r.sid=s.sid and b.bid=r.bid and s.age>=40 group by(r.bid) having count(distinct r.sid>=5);

7.   Create a view that shows the names and colours of all the boats that have been reserved by a sailor with a specific rating.
             create view sailors_with_specific_rating as select b.bname, b.color from boat b, sailor s, reserves r where r.sid=s.sid and r.bid=b.bid and s
.rating=9;

8.     A trigger that prevents boats from being deleted If they have active reservations.
            delimiter $ 
create trigger prevent_deletion_of_boats before delete on boat for each row begin declare num_reservations int; set num_reservations=(Select count(*) from reserves r where r.bid=old.bid); if num_reservations>0 then signal sqlstate '45000' set message_text = 'Cannot delete boats with active reservations!'; end if; end;$

delete from boat where bid=105;
