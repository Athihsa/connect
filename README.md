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

2. Find all sailor id’s of sailors who have a rating of at least 8 or reserved boat 103.
           (select s.sid from sailor s where s.rating>=8) UNION ( select sid from reserves r where bid=103);

3. Find the names of sailors who have not reserved a boat whose name contains the string “storm”. Order the names in ascending order.
           select s.sname from sailor s where s.sid not in (select r.sid from reserves r where r.bid in(select b.bid from boat b where b.bname like "%St
orm%")) Order by s.sname;

4. Find the names of sailors who have reserved all boats.
           select s.sname from sailor s where not exists( select * from boat b where not exists( select * from reserves r where s.sid=r.sid and b.bid=r.
bid));

5. Find the name and age of the oldest sailor.
             select s.sname, s.age from sailor s where s.age in (select max(s.age) from sailor s);

6. For each boat which was reserved by at least 5 sailors with age >= 40, find the boat id and the average age of such sailors.
            select b.bid, avg(s.age) from sailor s, boat b, reserves r where r.sid=s.sid and b.bid=r.bid and s.age>=40 group by(r.bid) having count(distinct r.sid>=5);

7. Create a view that shows the names and colours of all the boats that have been reserved by a sailor with a specific rating.
             create view sailors_with_specific_rating as select b.bname, b.color from boat b, sailor s, reserves r where r.sid=s.sid and r.bid=b.bid and s
.rating=9;

8. A trigger that prevents boats from being deleted If they have active reservations.
            delimiter $ 
            create trigger prevent_deletion_of_boats before delete on boat for each row begin declare num_reservations int; set num_reservations=(Select count(*) 
            from reserves r where r.bid=old.bid); if num_reservations>0 then signal sqlstate '45000' set message_text = 'Cannot delete boats with active  reservations!'; end if; end;$

 To show if the trigger is working ---> delete from boat where bid=105;







Insurance Database:person--->owns--->car:1:N    person--->participate--->car--->accident: 1:1:1

create table person(driverid varchar(20) primary key, name varchar(20), address varchar(40));
create table car(regno varchar(20) primary key, model varchar(20), year int);
create table accident(reportno int primary key, accidate date, location varchar(40));
create table owns(driverid varchar(20), regno varchar(20), foreign key(driverid) references person(driverid),foreign key(regno) references car(regno));
create table participated(driverid varchar(20), regno varchar(20), reportno int, amt int, foreign key(driverid) references person(driverid),
foreign key(regno) references car(regno), foreign key(reportno) references accident(reportno));

insert into person values('a01', 'Smith', 'London'),('a02','Ramesh','Kuvempunagar'),('a03','Srujan','Shivnagar'),('a04','Dhruvanth','KR Nagar'),('a05','Gopal','Siddarth Layout');
 insert into car values('KA09MA1231','Mazda',2020),('KA09MA1232','Wagnor',2012),('KA09MA1233','Santro',2010),('KA09MA1234','Benz',2015),('KA09MA1235','Swift Desire',2023);
insert into owns values('a01','KA09MA1231'),('a02','KA09MA1232'),('a03','KA09MA1233'),('a04','KA09MA1234'),('a05','KA09MA1235'),('a01','KA09MA1231');
insert into accident values(101, "2021-09-09",'Agra'),(102,"2022-12-12",'Ahmedabad'),(103,"2023-02-02","Banglore");
insert into participated values('a01','KA09MA1231',101,100000),('a03','KA09MA1233',103,30000),('a01','KA09MA1231',102,500000),('a01','KA09MA1231',103,90000);

1. Find the total number of people who owned cars that were involved in accidents in 2021. 
      select count(distinct o.driverid) from owns o join participated p on p.driverid=o.driverid and p.regno=o.regno join accident a on a.reportno=p.reportno where year(a.accidate)=2023;

2.  Find the number of accidents in which the cars belonging to “Smith” were involved.  
         select count(distinct pa.reportno) from owns o join participated pa on pa.regno=o.regno and pa.driverid=o.driverid join person p on p.driverid=o.driverid where p.name='Smith';

3.  Add a new accident to the database; assume any values for required attributes. 
     insert into accident values(104,"2022-02-26","Sigandur");

4. Delete the Mazda belonging to “Smith”.  
    delete from owns where driverid in (select p.driverid from person p where name='Smith') and regno in (select regno from car where model='Mazda');

5. Update the damage amount for the car with license number “KA09MA1234” in the accident with report. 
       update participated set amt=50000 where regno='KA09MA1234' and reportno=103;

6. A view that shows models and year of cars that are involved in accident:
     CREATE VIEW car_involved_in_accident AS SELECT model, year FROM car JOIN participated p ON car.regno = p.regno;

7. A trigger that prevents a driver from participating in more than 3 accidents in a given year.
     CREATE TRIGGER limit_accident BEFORE INSERT ON participated FOR EACH ROW BEGIN IF (SELECT COUNT(*) FROM participated WHERE driverid = NEW.driverid AND YEAR(accidate) = YEAR(CURRENT_DATE)) >= 3 THEN SIGNAL SQLSTATE ‘45000’ SET MESSAGE_TEXT = ‘Driver has exceeded the limit of accidents in a year’; END IF; END;
