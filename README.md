# normalization-7dbs

Authors: Richard Decal and Joe Comer

## Normalize the jail database:
  ### 1. ER diagram of important entities.
    
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


  ### 2. New db jail_norm_ComerDecal.db (readable by Noah + Dr Gillman), and create the tables for a jail db in 3rd normal form. Use all appropriate integrity constraints.
  
  ### 3. Populate the new db
   
      1. Open jail.db.
      2. Attach your new db, giving it a schema name of jailnew.
      3. Now you can use table foo in the new db, as jailnew.foo.
## 1. Put fanfiction.stories into 2nd normal form (which implies 1st normal form). To keep url as primary key you’ll need to create a new table. starringchars isn’t the only problem. You’ll probably want to read the Postgres docs on string functions. 

##2. Seven Databases, Chapter 2, Days 1 and 2 homework.
###   1. Section 2.2, Find problem 3.
In the addresses FOREIGN KEY, find in the docs what MATCH FULL means.

MATCH FULL will not allow one column of a multicolumn foreign key to be null unless all foreign key columns are null.

###   2. Section 2.2, Do problems 1, 3.
Select all the tables we created (and only those) from pg_class.

Write a query that finds the country name of the LARP Club event.
###   3. Section 2.3, Do problem 2.

A temporary table was not the best way to implement our event calendar
pivot table. The generate_series(a, b) function returns a set of records, from
a to b. Replace the month_count table SELECT with this.
34 • Chapter 2. PostgreSQL
