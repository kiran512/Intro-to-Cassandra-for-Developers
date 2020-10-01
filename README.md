## 🎓🔥 Intro to Cassandra for Developers using DataStax Astra 🔥🎓

Welcome to the 'Intro to Cassandra for Developers' workshop! In this two-hour workshop, the Developer Advocate team of DataStax shows the most important fundamentals and basics of the powerful distributed NoSQL database Apache Cassandra. Using Astra, the cloud based Cassandra-as-a-Service platform delivered by DataStax, we will cover the very first steps for every developer who wants to try to learn a new database: creating tables and CRUD operations. 

It doesn't matter if you join our workshop live or you prefer to do at your own pace, we have you covered. In this repository, you'll find everything you need for this workshop:

## Table of Contents

| Title  | Description
|---|---|
|
|
| **1. Create a table** | [Create a table](#2-create-a-table) |
| **2. Execute CRUD (Create, Read, Update, Delete) operations** | [Execute CRUD operations](#3-execute-crud-operations) |

[🏠 Back to Table of Contents](#table-of-contents)


## 1. Create a table
Ok, now that you have a database created the next step is to create a table to work with. 

**✅ Step 2a. Navigate to the CQL Console and login to the database**

In the Summary screen for your database, select **_CQL Console_** from the top menu in the main window. This will take you to the CQL Console with a login prompt.

<img width="1000" alt="Screenshot 2020-09-30 at 13 51 55" src="https://user-images.githubusercontent.com/20337262/94687448-2aff1c00-0324-11eb-8aa6-516185d01ce8.png">

Enter in the credentials we used earlier to create the **_killrvideo_** database. If you followed the instructions earlier this should be **_KVUser_** and **_KVPassword1_**. If you already created the **_killrvideo_** database at some point before this workshop and used different credentials, just use those instead.

<img width="1253" alt="Screenshot 2020-09-30 at 13 53 43" src="https://user-images.githubusercontent.com/20337262/94687593-613c9b80-0324-11eb-8db8-35a76a786b18.png">


**✅ Step 2b. Describe keyspaces and USE killrvideo**

Ok, you're logged in, and now we're ready to rock. Creating tables is quite easy, but before we create one we need to tell the database which keyspace we are working with.

First, let's **_DESCRIBE_** all of the keyspaces that are in the database. This will give us a list of the available keyspaces.

📘 **Command to execute**
```
desc KEYSPACES;
```
_"desc" is short for "describe", either is valid_

📗 **Expected output**

<img width="1000" alt="Screenshot 2020-09-30 at 13 54 55" src="https://user-images.githubusercontent.com/20337262/94687725-8cbf8600-0324-11eb-83b0-fbd3d7fbdadc.png">

Depending on your setup you might see a different set of keyspaces then in the image. The one we care about for now is **_killrvideo_**. From here, execute the **_USE_** command with the **_killrvideo_** keyspace to tell the database our context is within **_killrvideo_**.

📘 **Command to execute**
```
use songbrowser;
```

📗 **Expected output**

Notice how the prompt displays ```KVUser@cqlsh:killrvideo>``` informing us we are **using** the **_killrvideo_** keyspace. Now we are ready to create our table.

**✅ Step 2c. Create the tables**

At this point we can execute a command to create the tables for songbrowser sample. Just copy/paste the following command into your CQL console at the prompt.

📘 **Command to execute**

```
// producer
CREATE TYPE IF NOT EXISTS songbrowser.producer (
  name          TEXT,
  country_code  TEXT  
);

CREATE TABLE IF NOT EXISTS songbrowser.albums (
  album_title    VARCHAR,
  artist         VARCHAR,
  genre          VARCHAR,
  producer       producer,
  record_label   VARCHAR,
  release_year   INT,
  PRIMARY KEY ((artist), release_year, album_title)
);

CREATE TABLE IF NOT EXISTS songbrowser.songs_by_albums (
  album_title    VARCHAR,
  artist         VARCHAR,
  genre          VARCHAR,
  performers     LIST<VARCHAR>,
  release_year   INT,
  song_title     VARCHAR,
  track_no       INT,
  PRIMARY KEY ((release_year, album_title), track_no)
);

CREATE TABLE IF NOT EXISTS songbrowser.songs_by_artists (
  album_title    VARCHAR,
  artist         VARCHAR,
  genre          VARCHAR,
  performers     LIST<VARCHAR>,
  release_year   INT,
  song_title     VARCHAR,
  track_no       INT,
  PRIMARY KEY ((artist), release_year, album_title, track_no)
);

CREATE CUSTOM INDEX IF NOT EXISTS album_album_title_partial ON songbrowser.albums (album_title) USING 'org.apache.cassandra.index.sasi.SASIIndex'
  WITH OPTIONS = { 'mode': 'CONTAINS' };

CREATE MATERIALIZED VIEW IF NOT EXISTS songbrowser.albums_by_genre AS
  SELECT album_title, release_year, artist, genre
  FROM songbrowser.albums
  WHERE album_title IS NOT NULL and release_year IS NOT NULL and artist IS NOT NULL and genre IS NOT NULL
  PRIMARY KEY ((genre), release_year, artist, album_title);


```


Then **_DESCRIBE_** your keyspace tables to ensure it is there.

📘 **Command to execute**

```
desc tables;
```

📗 **Expected output**


Aaaand **BOOM**, you created a tables in your database. That's it. Now, we'll move to the next section in the presentation and break down the method used to create a data model with Apache Cassandra.

[🏠 Back to Table of Contents](#table-of-contents)

## 3. Execute CRUD operations
CRUD operations stand for create, read, update, and delete. Simply put, they are the basic types of commands you need to work with ANY database in order to maintain data for your applications.

Then **_DESCRIBE_** your keyspace tables to ensure they are both there.

📘 **Command to execute**

```
desc tables;
```


**✅ Step 3a. (C)RUD = create = insert data**

Our tables are in place so let's put some data in them. This is done with the **INSERT** statement. We'll start by inserting data into the tables.

📘 **Commands to execute**

```
INSERT INTO albums (artist, release_year, album_title, genre, record_label, producer)
VALUES ('Red Hot Chili Peppers', 2002, 'By the Way', 'Funk', 'WMG', {name: 'Warner Music GrouXp', country_code: 'us'});

INSERT INTO albums (artist, release_year, album_title, genre, record_label, producer)
VALUES ('System of a Down', 2005, 'Hypnotize', 'Hard rock', 'American', {name: 'American Records', country_code: 'us'});
INSERT INTO songs_by_albums (release_year, album_title, track_no, artist, genre, performers, song_title)
VALUES (2005, 'Mezmerize', 1, 'System of a Down', 'Hard rock', ['Tankian', 'Malakian', 'Odadjian', 'Dolmayan'], 'Soldier Side');

INSERT INTO songs_by_albums (release_year, album_title, track_no, artist, genre, performers, song_title)
VALUES (2005, 'Mezmerize', 2, 'System of a Down', 'Hard rock', ['Tankian', 'Malakian', 'Odadjian', 'Dolmayan'], 'BYOB');

INSERT INTO songs_by_albums (release_year, album_title, track_no, artist, genre, performers, song_title)
VALUES (2005, 'Mezmerize', 3, 'System of a Down', 'Hard rock', ['Tankian', 'Malakian', 'Odadjian', 'Dolmayan'], 'Revenga');

INSERT INTO songs_by_albums (release_year, album_title, track_no, artist, genre, performers, song_title)
VALUES (2005, 'Mezmerize', 4, 'System of a Down', 'Hard rock', ['Tankian', 'Malakian', 'Odadjian', 'Dolmayan'], 'Cigaro');

INSERT INTO songs_by_albums (release_year, album_title, track_no, artist, genre, performers, song_title)
VALUES (2005, 'Hypnotize', 1, 'System of a Down', 'Hard rock', ['Tankian', 'Malakian', 'Odadjian', 'Dolmayan'], 'Soldier Side');

INSERT INTO songs_by_artists (release_year, album_title, track_no, artist, genre, performers, song_title)
VALUES (2005, 'Mezmerize', 1, 'System of a Down', 'Hard rock', ['Tankian', 'Malakian', 'Odadjian', 'Dolmayan'], 'Soldier Side');

INSERT INTO songs_by_artists (release_year, album_title, track_no, artist, genre, performers, song_title)
VALUES (2005, 'Mezmerize', 2, 'System of a Down', 'Hard rock', ['Tankian', 'Malakian', 'Odadjian', 'Dolmayan'], 'BYOB');

INSERT INTO songs_by_artists (release_year, album_title, track_no, artist, genre, performers, song_title)
VALUES (2005, 'Mezmerize', 3, 'System of a Down', 'Hard rock', ['Tankian', 'Malakian', 'Odadjian', 'Dolmayan'], 'Revenga');

INSERT INTO songs_by_artists (release_year, album_title, track_no, artist, genre, performers, song_title)
VALUES (2005, 'Mezmerize', 4, 'System of a Down', 'Hard rock', ['Tankian', 'Malakian', 'Odadjian', 'Dolmayan'], 'Cigaro');

```

```

**✅ Step 3c. C(R)UD = read = read data**

Now that we've inserted a set of data, let's take a look at how to read that data back out. This is done with a **SELECT** statement. In its simplist form we could just execute a statement like the following **_**cough_** **_**cough_**:
```
SELECT * FROM albums;
```

You may have noticed my coughing fit a moment ago. Even though you can execute a **SELECT** statement with no partition key definied this is NOT something you should do when using Apache Cassandra. We are doing it here for illustration purposes only and because our dataset only has a handful of values. Given the data we inserted earlier a more proper statement would be something like:
```
SELECT * FROM albums 
  WHERE artist = 'System of a Down';
```

The key is to ensure we are **always selecting by some partition key** at a minimum.

Ok, so with that out of the way let's **READ** the data we _"created"_ earlier with our **INSERT** statements.

📘 **Commands to execute**

```
// Read all data from the songs_by_artists table
SELECT * FROM songs_by_artists;

// Read the from songs by albums table using release year and album title
SELECT * FROM songs_by_albums 
  WHERE release_year = 2005 AND album_title = 'Mezmerize';
```

