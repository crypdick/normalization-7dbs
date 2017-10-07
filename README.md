# normalization-7dbs

Authors: Richard Decal and Joe Comer

## Normalize the jail database:
   1. ER diagram of important entities.
    
    
-------------------------------------------------------------	

###The table bookings contains all the the attributes uniquely determined by the bookingNumber.
=>CREATE TABLE bookings(bookingNumber int PRIMARY KEY, Agency varchar, ABN int, Court varchar, releaseDate timestamp, releaseCode varchar,SOID int, FOREIGN KEY (SOID) REFERENCES people(SOID));

###A table for the names of all charges with a serial PK to avoid repetition in other tables.
=>CREATE TABLE charges(charge_id INTEGER PRIMARY KEY AUTOINCREMENT, charge_name VARCHAR UNIQUE);

###Ditto for addresses.
=>CREATE TABLE Addresses(Address_id INTEGER PRIMARY KEY AUTOINCREMENT, address VARCHAR UNIQUE);


###Ditto for ChargeType.
=>CREATE TABLE ChargeType(chargeType_id INTEGER PRIMARY KEY AUTOINCREMENT, chargeType VARCHAR UNIQUE);



###SOID uniquely identifies a person, which determines race, DOB, and all the other attributes in this table.
=>CREATE TABLE people(SOID INTEGER PRIMARY KEY, name VARCHAR, DOB TIMESTAMP, POB VARCHAR, Race CHAR(1) CHECK (Race IN ("A","B","I","U","W","a","b","w")), e CHAR(1) CHECK (e IN ("H","N","U","h","n","u")));

###Now begin inserting data, being sure to select DISTINCT when necessary to not throw constraint violations.
=>INSERT INTO addresses (address) SELECT DISTINCT address address FROM jailnew.bookingsB;


=>INSERT INTO bookings SELECT bookingNumber, Agency, ABN, Court, releaseDate, releaseCode, SOID FROM jailnew.bookingsB;


=>INSERT INTO people (SOID, name, DOB, POB, Race, e) SELECT * FROM (select SOID, name, DOB, POB, race, e from jailnew.bookingsB group by SOID);


=>INSERT INTO ChargeType(chargeType) SELECT DISTINCT chargeType chargeType FROM jailnew.booking_addl_charge;

###This command makes sure we didn't miss any chargetypes that might have appeared in booking_addl_charges.
=>INSERT INTO ChargeType(chargeType) SELECT DISTINCT chargeType chargeType FROM jailnew.bookingsB WHERE jailnew.bookingsB.chargeType NOT IN (select chargeType from ChargeType);


=>INSERT INTO charges (charge_name) SELECT DISTINCT charge charge_name FROM (SELECT DISTINCT charge FROM jailnew.booking_addl_charge);


=> CREATE TABLE Courts(court_id INTEGER PRIMARY KEY AUTOINCREMENT, Court VARCHAR UNIQUE);


=>INSERT INTO Courts (Court) SELECT DISTINCT court court FROM jailnew.bookingsB;

###There are entire repeated rows in booking_addl_charges. bookingNumber, caseNumber--everything. I took this to mean that one person was being tried for multiple counts of the same crime in the same case on the same booking. I created a unique index for each count of each crime.
=>CREATE TABLE all_counts(bookingNumber INTEGER, chargeType_id INTEGER, charge_id INTEGER, caseNumber VARCHAR, FOREIGN KEY (bookingNumber) REFERENCES bookings(bookingNumber), FOREIGN KEY (charge_id) REFERENCES charges(charge_id), FOREIGN KEY (chargeType_id) REFERENCES ChargeType(chargeType_id));

###Now I make a temporary table to consolidate some attributes that appear in separate tables in order to avoid multiple joins in one query. This temp table was too big for ROM though, so I made it a not-temparary table, and then drop it when I'm done with it.
=>CREATE TABLE all_charges_extra(bookingNumber INTEGER, chargeType VARCHAR, chargeType_id INTEGER, charge VARCHAR, charge_id INTEGER, caseNumber VARCHAR);


###Now we grab all the data we can in one query and insert it into all_counts. I unioned bookingsB and booking_addl_charges to grab all counts at once. But I need separate queries to efficiently grab the charge_id without doing a possibly hazardous triple join.
=>INSERT INTO all_counts (bookingNumber, charge_id, caseNumber) SELECT b.bookingNumber bookingNumber, c.charge_id charge_id, b.caseNumber caseNumber FROM all_charges_extra b LEFT JOIN charges c ON b.charge = c.charge_name;


###Last missing column from the all_counts table is the charge_id. 
=>INSERT INTO all_counts(charge_id) SELECT c.charge_id charge_id FROM all_charges_extra a LEFT JOIN charges c ON a.charge=c.charge_name;










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