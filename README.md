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

#  normalize the fanfiction db
## 1. Put fanfiction.stories into 2nd normal form (which implies 1st normal form). To keep url as primary key you’ll need to create a new table. starringchars isn’t the only problem. You’ll probably want to read the Postgres docs on string functions.

    fanfiction_duffrindecal=> \d  stories_orig;
                                           Table "public.stories_orig"
        Column     |            Type             |                         Modifiers
    ---------------+-----------------------------+-----------------------------------------------------------
     rating        | character varying(2)        |
     updated       | timestamp without time zone |
     favorites     | integer                     |
     starringchars | text                        |
     chapters      | integer                     |
     complete      | boolean                     |
     collectedinfo | text                        |
     genre         | text                        |
     description   | text                        |
     language      | text                        |
     author        | text                        |
     url           | text                        | not null
     follows       | integer                     |
     title         | text                        |
     reviews       | integer                     |
     published     | timestamp without time zone |
     words         | integer                     |
     id            | integer                     | not null default nextval('stories_orig_id_seq'::regclass)
    Indexes:
        "stories_orig_pkey" PRIMARY KEY, btree (id)
        "stories_orig_url_idx" btree (url)


In order to get this into 2nd normal form, we need to unnest all our fields, and we must have all non-primary fields be dependent (either directly or indirectly) on the primary key of the table.

The fields we need to unnest are the starring characters and genres. To do this, we make put characters and genres into their own tables. To relate our stories table to the characters and genres, we make a starring_characters and genreness table:

Technically speaking, we could assign URL as our primary key and stay in 2nd normal form. However, that would be cumbersom. I am moving URLs to their own table and using url_id as the primary key in stories:

    CREATE TABLE urls AS (SELECT DISTINCT url FROM stories_orig);

    ALTER TABLE urls ADD COLUMN url_id SERIAL;

Creating genres: TODO: give proper names

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


Now, we will replace URL in our stories table with url_id:

CREATE TABLE stories AS (
SELECT urls.url_id, rating , updated, favorites,  chapters, complete, collectedinfo, description, language, author, follows, title, reviews, published, words
FROM stories_orig
JOIN urls ON urls.url = stories_orig.url
);



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

create db
sql attach database book as b;

string to array
select unnest(string_to_array(genre,'/')) from stories
select distinct
Unnest expand an array to a set of rows
CREATE TABLE newtest (id serial primary key, num integer, data varchar)
