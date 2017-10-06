# normalization-7dbs

Authors: Richard Decal and Joe Comer

## Normalize the jail database:
  ### 1. ER diagram of important entities.

![](bookings_ER_Diagram.png)  
  
  ### 2. New db jail_norm_ComerDecal.db (readable by Noah + Dr Gillman), and create the tables for a jail db in 3rd normal form. Use all appropriate integrity constraints.

  ### 3. Populate the new db

      1. Open jail.db.
      2. Attach your new db, giving it a schema name of jailnew.
      3. Now you can use table foo in the new db, as jailnew.foo.

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


1. Put fanfiction.stories into 2nd normal form (which implies 1st normal form). To keep url as primary key you’ll need to create a new table. starringchars isn’t the only problem. You’ll probably want to read the Postgres docs on string functions. 



#  normalize the fanfiction db
## 1. Put fanfiction.stories into 2nd normal form (which implies 1st normal form). To keep url as primary key you’ll need to create a new table. starringchars isn’t the only problem. You’ll probably want to read the Postgres docs on string functions.


We need to unnest the starring characters and genres. To do this, we put characters and genres into their own tables. To relate our stories table to the characters and genres, we make a starring_characters and genreness table:

Technically speaking, we could assign URL as our primary key and stay in 2nd normal form. However, that would be cumbersome. I am moving URLs to their own table and using url_id as the primary key in stories:

    CREATE TABLE urls AS (SELECT DISTINCT url FROM stories_orig);

    ALTER TABLE urls ADD COLUMN url_id SERIAL;

Creating genres:

    CREATE TABLE genres AS (SELECT DISTINCT  unnest(string_to_array(genre,'/')) genre FROM stories_orig);

    ALTER TABLE genres  ADD COLUMN genre_id SERIAL;

    ALTER TABLE genres ADD PRIMARY KEY (genre_id);

Creating characters:

    CREATE TABLE characters AS (SELECT DISTINCT unnest(string_to_array(rtrim(starringchars), ' & ')) character_name FROM stories_orig);

    ALTER TABLE characters ADD COLUMN character_id SERIAL;

    ALTER TABLE characters ADD PRIMARY KEY (character_id);

Creating starringchars:

    CREATE TABLE starringchars AS (SELECT url url_id, unnest(string_to_array(rtrim(starringchars), ' & ')) character_id FROM stories_orig);

Replace URL string with the url_id we made earlier

    UPDATE starringchars SET url_id = urls.url_id
    FROM urls
    WHERE urls.url = starringchars.url_id;

Replace character names with the character_id we made earlier:

    UPDATE starringchars SET character_id = characters.character_id
    FROM characters
    WHERE starringchars.character_id = characters.character_name;

Creating genreness:

    CREATE TABLE genreness AS (SELECT url, unnest(string_to_array(genre,'/')) genreness FROM stories_orig);

Replace URLs with their id:

    UPDATE genreness set url = urls.url_id
    from urls
    where urls.url = genreness.url;

Replace genres with their ID:

    UPDATE genreness SET genreness = genres.genre_id
    FROM genres
    WHERE genres.genre = genreness.genreness;


Finally, we will replace URL in our stories table with url_id:

    CREATE TABLE stories AS (
    SELECT urls.url_id, rating , updated, favorites,  chapters, complete, collectedinfo, description, language, author, follows, title, reviews, published, words
    FROM stories_orig
    JOIN urls ON urls.url = stories_orig.url
    );


## Seven Databases, Chapter 2, Days 1 and 2 homework.
###   1. Section 2.2, Find problem 3.
   
MATCH FULL is a restraint on foreign keys consisting of more than one column, establishing that either both columns must have non-null values, or both must have null values.

###   2i

     SELECT relname FROM pg_class WHERE relname IN ('countries','cities','venues','events');
### 2ii

    SELECT c.country_name FROM venues v JOIN countries c ON  v.country_code=c.country_code JOIN events e ON v.venue_id = e.venue_id;
    
###3iii
	ALTER TABLE venues ADD active boolean default TRUE;

### 3. Section 2.3, Do problem 2.
   
        SELECT * FROM crosstab( 'SELECT extract(year from starts) AS year, extract(month from starts) AS month, count(*) FROM events GROUP BY year, month',;
        'SELECT * FROM generate_series(1,12)' ) AS ( year int, jan int, feb int, mar int, apr int, may int, jun int, jul int, aug int, sep int, oct int, nov int, dec int ) 
        ORDER BY YEAR;

