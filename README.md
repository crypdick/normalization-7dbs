# normalization-7dbs

Authors: Richard Decal and Joe Comer

## Normalize the jail database:
   1. ER diagram of important entities.
    
    sqlite> .schema
    CREATE TABLE booking_addl_charge(
    "bookingNumber" TEXT,
    "chargeType" TEXT,
    "charge" TEXT,
    "court" TEXT,
    "caseNumber" TEXT
    );
    CREATE TABLE booking_charges(
    bookingNumber TEXT,
    charge TEXT
    );
    CREATE TABLE "bookingsA"(
    bookingNumber TEXT,
    name TEXT,
    race TEXT,
    sex TEXT,
    DOB TEXT,
    arrestDate TEXT,
    bookingDate TEXT,
    releaseDate TEXT,
    releaseCode TEXT,
    releaseRemarks TEXT
    );
    CREATE TABLE "bookingsB" ("name" TEXT,
    "bookingNumber" TEXT,
    "agency" TEXT,
    "ABN" TEXT,
    "race" TEXT,
    "sex" TEXT,
    "e" TEXT,
    "DOB" TEXT,
    "chargeType" TEXT,
    "charge" TEXT,
    "court" TEXT,
    "caseNumber" TEXT,
    "address" TEXT, city text,  "POB" TEXT,
    "releaseDate" TEXT,
    "releaseCode" TEXT,
    "SOID" TEXT
    );
    CREATE TABLE homeless_addresses(
    "address" TEXT,
    "rank" TEXT,
    "description" TEXT
    );
    CREATE TABLE homeless_charges(
    "charge" TEXT,
    "rank" TEXT
    );
    CREATE TABLE inmates(
    SOID int PRIMARY KEY,
    name text,
    DOB text,
    sex text,
    race text,
    e text);
    CREATE TABLE variables(x);
    CREATE INDEX booking_address on bookingsB (address);
    CREATE INDEX booking_charge on bookingsB (charge);
    CREATE INDEX booking_soid on bookingsB (bookingNumber,SOID);
    CREATE INDEX bookingsA1_booking on "bookingsA" (bookingNumber);
	
-------------------------------------------------------------	
CREATE TABLE bookings(bookingNumber int PRIMARY KEY, Agency varchar, ABN int, Court varchar, releaseDate timestamp, releaseCode varchar,SOID int, foreign key(SOID) references people(SOID));
CREATE TABLE charges(charge_id integer primary key autoincrement, charge_name varchar unique);
CREATE TABLE Addresses(Address_id integer primary key autoincrement, address varchar unique);
CREATE TABLE people(SOID int primary key, name varchar, DOB timestamp, POB varchar, Race char(1) check (Race in ("A","B","I","U","W","a","b","w")), e char(1) check (e in ("H","N","U","h","n","u")));
CREATE TABLE BookingsCharges(bookingNumber int foriegn key references bo                                                                             okings(bookingNumber), chargeType varchar, charge_id foriegn key references char                                                                             ges(charge_id), primary key (bookingNumber,charge_id));
insert into addresses values(address) select distinct address address from jailnew.bookingsB;
insert into bookings select bookingNumber, Agency, ABN, Court, releaseDate, releaseCode, SOID from jailnew.bookingsB;
insert into people (SOID, name, DOB, POB, Race, e) select * from (select SOID, name, DOB, POB, race, e from jailnew.bookingsB group by SOID);
insert into BookingsCharges(bookingNumber, chargeType, charge_id) select bookingNumber, chargeType, charge_id from (select a.bookingNumber, a.chargeType, b.charge_id from jailnew.booking_addl_charge a left join charges b on a.charge = b.charge_name);
insert into BookingsCharges(bookingNumber, chargeType, charge_id) select bookingNumber, chargeType, charge_id from (select a.bookingNumber, a.chargeType, b.charge_id from jailnew.bookingsB a left join charges b on a.charge = b.charge_name);

	

   2. New db jail_norm_ComerDecal.db (readable by Noah + Dr Gillman), and create the tables for a jail db in 3rd normal form. Use all appropriate integrity constraints.
   3. Populate the new db
   
      1. Open jail.db.
      2. Attach your new db, giving it a schema name of jailnew.
      3. Now you can use table foo in the new db, as jailnew.foo.
1. Put fanfiction.stories into 2nd normal form (which implies 1st normal form). To keep url as primary key you’ll need to create a new table. starringchars isn’t the only problem. You’ll probably want to read the Postgres docs on string functions. 
2. Seven Databases, Chapter 2, Days 1 and 2 homework.
   1. Section 2.2, Find problem 3.
Match Full is a restraint on foreign keys consisting of more than one column, establishing that either both columns must have non-null values, or both must have null values.
   2. Section 2.2, Do problems 1, 3.
i.	1. select relname from pg_class where relname in ('countries','cities','venues','events');
ii.	2. select c.country_name from venues v join countries c on  v.country_code=c.country_code join events e on v.venue_id = e.venue_id;
iii.	3. alter table venues add active boolean default TRUE;

   3. Section 2.3, Do problem 2.
SELECT * FROM crosstab( 'SELECT extract(year from starts) as year, extract(month from starts) as month, count(*) FROM events GROUP BY year, month', 
'SELECT * FROM generate_series(1,12)' ) AS ( year int, jan int, feb int, mar int, apr int, may int, jun int, jul int, aug int, sep int, oct int, nov int, dec int ) 
ORDER BY YEAR;