# Object Relational Mapping 

This is a teachnique of accessing a relational database 
It uses an Objected Oriented Programming Language

It's a way for our Ruby programs to manage database data by "mapping" database tables 
to classes and instances of classes to rows in those tables. 

This is an example of code that connects your Ruby program to a given database

# database_connection = SQLite3::Database.new('db/my_database.db')
# database_connection.execute("Some SQL Statement")

When "mapping" our program to a database, we equate: 
    Classes with Database Tables
    Instances of Those Classes with Table Rows 
We write Ruby code that "wraps" or handles SQL 

# Pets Database 


database_connection = SQLite3::Database.new('db/pets.db')

We would then create an owners table and a cats table: 

    database_connection.execute(
        "CREATE TABLE IF NOT EXISTS cats(
            id INTEGER PRIMARY KEY,
            name TEXT,
            breed TEXT, 
            age INTEGER
        )"
    )

    database_connection.execute(
        "CREATE TABLE IF NOT EXISTS owners(
            id INTEGER PRIMARY KEY,
            name TEXT,
            breed TEXT, 
            age INTEGER
        )"
    )

Then we insert new Cats and Owners into these tables: 

    database_connection.execute("INSERT INTO cats (name, breed, age) VALUES ("Maru", "Scottish Fold", 3)")
    database_connection.execute("INSERT INTO cats (name, breed, age) VALUES ("Hana", "Tortoiseshell", 1)")

We can write a .save method on our Cats class that handles the common action of INSERTING data into the database. 

class Cat 
    @@all = [] 

    def initialize(name, breed, age)
        @name = name 
        @breed = breed 
        @age = age 
        @@all = self 
    end

    def self.all 
        @@all 
    end

    def self.save(name, breed, age, database_connection)
        database_connection.execute("INSERT INTO (name, breed, age) VALUES (?,?,?)", name, breed, age)
    end 
end 

# Now let's create some new cats and save them to the database:

database_connection = SQLite3::Database.new('db/pets.db') 

Cat.new("Maru", "Scottish Fold", 3)
Cat.new("Hana", "Tortoiseshell", 1)

Cat.all.each do |cat|
    Cat.save(cat.name, cat.breed, cat.age, database_connection)
end

Inside this iteration, we use the Cat.save method
We give it arguments of the data specific to each cat to INSERT those cat records into the cats table. 


# ORM: Mapping Ruby Classes To Database Tables 

We are building a music play that will allow users to store music and browse their songs. 
It will have a Song class. 
Each Song instance will have a name and an album attribute. 


class Song 
    attr_accessor :name, :album 

    def initialize(name, album)
        @name = name 
        @album = album
    end

end 

We have an attr_accessor for name and album. 
In order to "map" this Song Class to a songs Database Table, we need to create our Database. 
We also need to create our Songs Table. 

It is conventional to Pluralize the name of the class to create the name of the table. 
Song Class -> "songs" table 

# Creating The Database 
It is not the responsibility of our Song class to create the database. 
Classes are mapped to tables insides a database, not to the database as a whole. 

It is the responsibility of our program as a whole to create and establish the database. 
Our programs will have a config directory that contains an environment.rb  
Example: 

# require 'sqlite3'
# require_relative '../lib/song.rb'
# DB = {:conn => SQLite3::Database.new("db/music.db")}

This will create a new database called music.db that is stored inside the db subdirectory of our app
It will return a Ruby object that represents the connection between our Ruby program and our newly created SQL Database. 

This is what gets returned by the line of code above 

#<SQLite3::Database:0x007f9d6c294508
 @authorizer=nil,
 @busy_handler=nil,
 @collations={},
 @encoding=nil,
 @functions={},
 @readonly=false,
 @results_as_hash=nil,
 @tracefunc=nil,
 @type_translation=nil>

This object is created for us by the code provided by the SQLite-Ruby gem. 
This is the object that connects the rest of our Ruby program. 

# For Example: 
Any code we write to create artists, songs, and genres to our SQL Database. 

# Note 
There are a number of methods available to us that are provided by the SQLite-Ruby gem
We can call on the above object to execute commands against our database. 

# DB[:conn]
We set up a Constant, DB that is equal to a Hash. It contains our connection to the Database. 
In our lib/song.rb file, we can access the DB contstant and the database connection. 


# Creating The Table 
We write a class method in our Song class that will create a table. 
To "map" our class to a Database Table, we will create a table with the same name as our class 
We will also give that table Column Names That Match The "attr_accessors" Of Our Class. 

# Example 

class Song 
    attr_accessor :name, :album, :id 

    def initialize(name, album, id=nil)
        @id = id 
        @name = name 
        @album = album 
    end

    def self.create_table
        "CREATE TABLE IF NOT EXISTS(
            id INTEGER PRIMARY KEY, 
            name TEXT,
            album TEXT
        )"
    DB[:conn].execute(sql) 
    end 
end 

# EXPLANATION 

# id 
We are initializing an individual Song instance with an id attribute that has a default value of nil. 
Songs need an id attribute only because they will be saved into the database. 
Each table row needs an id value which is the primary key. 

When we create a new song with the Song.new method, we do not set that song's id. 
A song gets an id only when it gets saved into the database

We set the default value of the id argument to nil. 
This will allow us to create new iinstances that do not have an id value. 
In a relational database, the id of a given record must be unique. 

# .create_table 

We created this class method that crafts an SQL statement to create a songs table 
It gives the table column names that match the attributes of an individual instance of a Song.

It is a class method because it is the job of the Class as a whole to create the table that is mapped to. 

# Mapping Class Instances To Table Rows 
We are not saving Ruby objects in our database. 
We are going to take the individual attributes of a given instance song's name and album and save those attributes that describe an individual song to the database as one, single row. 


# Example

gold_digger = Song.new("Gold Digger", "Late Registration") 

gold_digger.name  => "Gold Digger"
gold_digger.album => "Late Registration" 

There are two attributes, name and album set equal to the above values. 
In order to save the song gold_digger into the songs table, we will use the name and album of the song to create a new row in that tabale. 

The SQL statement we want to execute will look like this: 

INSERT INTO songs (name, album) VALUES ("Gold Digger", "Late Registration")

# Saving Additional Songs 

hello = Song.new("Hello", "25")
hello.name => "Hello"
hello.album => "25" 

In order to Save hello into our Database, we do not insert the Ruby Object stored in the Hello variable. 
Instead, we use hello's name and album values to create a new row in the songs table.

INSERT INTO songs (name, album) VALUES ("Hello", "25")

# Notes 
We are repeating the same exact steps and using the same code. 
The only difference are the values that we are inserting into our songs table. 
We will abstract this functionality into an instance method, #save

# Inserting Data Into A Table With The #save Method
This method saves a given instance of our Song class into the songs tbale of our database. 

class Song
  sql = "INSERT INTO songs (name, album)  VALUES (?, ?)"
  DB[:conn].execute(sql, self.name, self.album)
end 


# Explained 
In order to INSERT data into our songs table, we need to craft an SQL INSERT Statement. 

# BOUND PARAMETERS 
Bound Parameters protects our program from getting confused by SQL injections and special characters. 
Instead of interpolating variables into a string of SQL, we are using the ? characters as place holders. 

# The SQlite3 Ruby Gem
This has an #execute method that takes values that we pass in as arguments and applies it to the values of the question marks.  

# Notes 
Our # save method inserts a record into our database that has the name and album values of the song instance we're trying to save. 
We are not saving the Ruby Object itself. 
We are creating a new row in our songs table that has the values that characterize that song instance. 

# IMPORTANT 
We do no tinsert an ID number. 
The INTEGER PRIMARY KEY datatype will assign and auto-increment the id attribute of each record that gets saved. 

# Creating Instances vs. Creating Table Rows 
The #new method creates a new instance of the song class, it is a new ruby object. 
The #save method takes the attributes that characterize a given song and saved them in a row of the songs table in our database. 

We keep the two methods separate because it's not a great idea to force objects to be saved every time they are created. 

class Song 
  attr_accessor :name, :album, :id 

  def initialize(name, album, id=nil)
    @id = id 
    @name = name 
    @album = album
  end 

  def self.create_table
    sql = "CREATE TABLE IF NOT EXISTS songs(
      id INTEGER PRIMARY KEY, 
      name TEXT, 
      album TEXT, 
    )"
    DB[:conn].execute(sql)
  end

  def save
    sql = "INSERT INTO songs (name, album) VALUES (?, ?)" 
    DB[:conn].execute(sql, self.name, self.album) 
  end
end   

# Now we can create and save songs 

Song.create_table 
hello = Song.new("Hello", "25")
ninety_nine_problems = Song.new("99 Problems", "The Black Album")

hello.save 
ninety_nine_problems.save 

# Giving Our Song Instance An id 
When we INSERT the data concerning a particular Song instance into our database table, we create a new row in that table. 
That row would look like this: 

ID	Name	Album
1	Hello	25

# Notes 
The database table's row has a column for Name, Album and also ID. 
Recall that we created our table to have a column for the primary key, ID, of a given record. 
As each record gets inserted into the database, it is given an ID number automatically.

The hello instance is stored in the database with the name and album that we gave it, plus an ID number that the database assigns to it.

We want our hello instance to completely reflect the database row it is associated with. 
This is so that we can retrieve it from the table later on with ease. 
Once the new row with hello's data is inserted into the table, let's grab the ID of that newly inserted row and assign it to be the value of hello's id attribute.

class Song

  attr_accessor :name, :album, :id
  
  def initialize(name, album, id=nil)
    @id = id
    @name = name
    @album = album
  end

  def save
    sql = "INSERT INTO songs (name, album) VALUES (?, ?)"
    DB[:conn].execute(sql, self.name, self.album)
    @id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs")[0][0] 
  end 

# Notes 
At the end of our save method, we use an SQL Query to grab the value of the ID Column of the last inserted row 
We set that equal to the given song instance's id attribute. 

# Instantiating a new instance of the Song Class
Inserting a new row into the database table that contains the information regarding that instance. 
Grabbing the ID of the newly inserted row 
Assigning that given Song's instance's id attribute equal to the ID of it's associated database table row. 

Song.create_table
hello = Song.new("Hello", "25")
ninety_nine_problems = Song.new("99 Problems", "The Black Album")

hello.save
ninety_nine_problems.save 

Create the songs table
Create two new song instances 

Use the song.save method to persist them to the database. 

We first create the new song and then we save it. 
This is done every time we want to create and save a song. 

This is Repetitive and Tedious. 

Creating an object and then saving a record object is common
We will write a method that does only that

# The .create Method 
This method will wrap the code we used above to create a new Song instance and save it.

class Song 

  .......

  def self.create(name, album)
    song = Song.new(name, album)
    song.save
    song 
  end 
end 

We use keyword argument to pass a name and album into our .create method
We use that name and album to instantiate a new song
Then, we use the #save method to persist that song to the database

The return value of .create should always be the object we created. 

song = Song.create(name: "Hello", album: "25")
# => #<Song:0x007f94f2c28ee8 @id=1, @name="Hello", @album="25">

song.name
# => "Hello"

song.album
# => "25"

# IMPORTANT 
We are not saving Ruby Objects into the Database
We are using the attributes of a given Ruby Object to create a new row in our database table. 



# MAPPING TABLES TO OBJECTS 

Ruby understands objects. 
The database understands raw data. 

We don't store Ruby objects in the database 
We don't get Ruby objects back from the database

We store raw data describing a given Ruby object in a table row
When we want to reconstruct a Ruby object from the stored data, we select that same row in the table. 

When we query the database, we write the code that takes data and turns it back into an instance of the appropriate class. 
We translate raw data that the database sends into Ruby objects that are instances of a particular class. 

# Example 

class Song(title, length) 
 - Responsible for making songs
 - Every song will come with two attributes a title and a length 
 - We need to look at all the songs that have already been created  
end 

# .new_from_db 
First, we need to convert what the DB gives us into a Ruby Object
We use this to create all of the Ruby Objects in our next two methods 

SQLite will return an array of data for each row

# For example
A row for Michael Jackson's "Thriller" (365 seconds long) that has a db id of 1 would look like

[1, "Thriller", 356] 

def self.new_from_db(row)
  new_song = Song.new 
  new_song.id = row[0]
  new_song.name = row[1]
  new_song.length = row[2]
  new_song  (This returns the newly created instance)
end 

# Note 
Since we're retrieving data from a DB we're using new. 
We don't need to create records. 
With this method we're reading data from SQLite and temporarily representing that data in Ruby. 


# Song.all 

Now we can write methods to retrieve data. 

sql = SELECT * FROM songs 

Then we make a call to our database using DB[:conn] 
This is a DB Hash that is located in config/environment.rb   DB = {:conn => SQLite3::Database.new("db/songs.db")}
The value of the hash is a new instance of the SQLite3::Database class

This is how we connect to the DB. 
Our DB Instance responds to a method called execute that acceps raw SQL as a string. 

class Song 
  def self.all 
    sql = SELECT * FROM songs 
    DB[:conn].execute(sql)
  end 
end 

This will return an array of rows from the database that matches our query. 
Now we have to iterate over each row and use self.new_from_db to create a new Ruby Object for each row. 

class Song 
  def self.all 
    sql = SELECT * FROM songs 
    DB[:conn].execute(sql).map do |row|
      self.new_from_db(row)
    end 
  end 
end 

# Song.find_by_name
This is similar to Song.all with an exception being that we have to include a name in our SQL Statement. 
To do this, we use a ? where we want the name parameter to be passed in and we include name as the second argument to the execute method:  

class Song
  def self.find_by_name(name)
    sql = SELECT * FROM songs WHERE name = ? LIMIT 1
    DB[:conn].execute(sql, name).map do |row|
      self.new_from_db(row) 
    end.first
  end 
end 


# Note 
The .first method is chained to the end of the DB[:conn].execute(sql, name).map block
The return value of the .map method is an array. 
We are retrieving the .first element from the returned array. 




# UPDATING RECORDS IN AN ORM 

Correct Order
1. Find the appropriate record 
2. Make the necessary changes 
3. Save again 


In the Ruby ORM attributes of a given object are stored as rows in a database table. 
We need to retrieve these attributes, reconstitute them into a Ruby object and make changes to that object 
using Ruby Methods. Then save the attributes back to the DB. 

# Updating A Record In A Ruby ORM 

class Song 
  attr_accessor :name, :album 
  attr_reader :id 

  def initialize(id=nil, name, album)
    @id = id
    @name = name 
    @album = album 
  end

  def self.create_table 
    sql = "CREATE TABLE IF NOT EXISTS songs (
      id INTEGER PRIMARY KEY,
      name TEXT, 
      album TEXT
    )"
    DB[:conn].execute(sql)
  end

  def save
    sql = INSERT INTO songs (name, album) VALUES (?, ?)
    DB[:conn].execute(sql, self.name, self.album)
    @id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs")[0][0]
  end 

  def self.create(name:, album:)
    song = Song.new(name, album)
    song.save
    song
  end

  def self.find_by_name(name)
    sql = "SELECT * FROM songs WHERE name = ?"
    result = DB[:conn].execute(sql, name)[0]
    Song.new(result[0], result[1], result[2])
  end 
end 

# Notes
With the Song class as defined above, we can create new Song instances, save them into the DB and retrieve them from the database. 

ninety_nine_problems = Song.create(name: "99 Problems", album: "The Blueprint")

Song.find_by_name("99 Problems")
# => #<Song:0x007f94f2c28ee8 @id=1, @name="99 Problems", @album="The Blueprint">

# Updating Songs

ninety_nine_problems.album
# => "The Blueprint"

ninety_nine_problems.album = "The Black Album"

ninety_nine_problems.album 
# => "The Black Album"

# Saving the record back into the DB 

UPDATE songs SET album = "The Black Album" WHERE name="99 Problems"

sql = "UPDATE songs SET album = ? WHERE name = ?"

DB[:conn].execute(sql, ninety_nine_problems.album, ninety_nine_problems.name)


# If We Want To Update Other Attributes 

Song.create(name: "Hella", album: "25")   

hello = Song.find_by_name("Hella")

sql = "UPDATE songs SET name = 'Hello' where name = ?"

DB[:conn].execute(sql, hello.name)



# update Method 

class Song 
  ... 

  def update
    sql = "UPDATE songs SET name = ?, album ?, WHERE name = ?"
    DB[:conn].execute(sql, self.name, self.album, self.name)
  end 
end 

# Update A Song 

hello = Song.create(name: "Hella", album: "25")
hello.name = "Hello"
hello.update 

# Notes 
How can the #update method use WHERE name = "#{self.name}" if we jusst changed the name of the song. It can't. 
The above #update method won't work if we're trying to update the name of a song. 

# Example 

sql = "UPDATE songs SET name = ?, album = ?, WHERE name = ?"
DB[:conn].execute(sql, self.name, self,album, self.name)

# Interpretation 
DB[:conn].execute(sql, "Hello", "25", "Hello")

Our database row still has a value of "Hella" in the "name" column 
Our query will fail to find the correct record and will not be able to upgrade it. 

We need to use the Primary Key ID of a database record and the id attribute of it's analogous Ruby Object

# Identifying Objects and Records Using ID 
We need a way to select a Ruby's Object analogous table row using some fixed and unique attribute.
Song records in the DB Table have a unique id. 
Our Song instances have an id attribute. 

We have been setting the id attribute of individual songs directly after the data regarding that song gets inserted into the DB Table
We have done this at the end of our #save meethod. 

The unique id number of a Song instance should come from the DB
When a song record gets inserted into the DB, that row automatically gets assigned a unique number. 
We need to grab that ID number from the DB Record and Assign it to the Song's instance id attribute. 



# Diagram 
1. We create a new instance of the Song Class. 
2. This instance has a name and album attribute. It's id id attribute is nil. 
3. The name and album of this Song Instance are used to create a new DB Record, which is a new row in the songs table. 
4. That record has an ID of 1, this would appear to be the first song we have ever saved in our DB. 
5. The ID of the newly created DB Record is then taken and assigned to the ID Attribute of the Original Song Object  

With this pattern, every instance of the Song Class that is ever saved into the DB will be assigned a unique id attribute
We can use this unique id attribute to differentiate it from the other Song objects we created
We can also use it to Find, Retrieve and Update unique songs

# Assigning Unique IDs on #save 
We should assign a unique id to a Song Instance right after we INSERT IT into the database. 
This is when it's equivalent database record will have a unique ID in the ID column. 
We want to grab the ID and use it to assign the Song object it's id value. 

# Note 
When do we INSERT a new record into our database? In the #save Method 

def save
  sql = "INSERT INTO songs (name, album) VALUES (?, ?)"
  DB[:conn].execute(sql, self.name, self.album)
end 

# Assign  
Right after we execute the SQL INSERT statement
We assign our Song object it's unique id from the DB

# How To Get The Unique ID Of A Record We Just Created?
We query the DB Table for the ID of the last inserted row: 

SELECT last_insert_rowid() FROM songs  

DB[:conn].execute("SELECT last_insert_rowid() FROM songs") 
# => [[1]]

# Note 
Whenever we execute SQL Statements against our DB using SQLite3 Ruby gem's #execute method we will get back an array of arrays. 

# Combining 

def save 
  sql = INSERT INTO songs (name, album) VALUES (?, ?)
  DB[:conn].execute(sql, self.name, self.album)
  @id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs")[0][0]
end 

hello = Song.create(name: "Hello", album: "25")

hello.name
# => "Hello"

hello.album
# => "25"

hello.id
# => 1


# Note 
Now our Song objects will get assigned a unique id attribute when they are saved the the DB
This means we can refactor our #update method such that it will only update the correct, unique record 


# Using id To Update Records 

Our #update method should identify the correct record to update
It should to based on the unique ID that both the Song Ruby Object and the Songs Table Row share. 

class Song 
  def update 
    sql = "UPDATE songs SET name = ?, album = ?, WHERE id = ?"
    DB[:conn].execute(sql, self.name, self.album, self.id)
  end 
end 

# Note 
No more updating the wrong record or being unable to find a record once we change it's name. 

# Refactoring The #save Method To Avoid Duplication 

CURRENTLY 

  def save
    sql = "INSERT INTO songs (name, album)  VALUES (?, ?)"
    DB[:conn].execute(sql, self.name, self.album)
    @id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs")[0][0] 
  end 

# Note
This method will always INSERT a new row into the DB table. 
What if we accidentally call #save on an object that has already been persisted and has an analogous database row? 
This would have the effect of creating a new DB row with the same attributes as an existing row. 
The only different would be the id number. 

hello = Song.new("Hello", "25")
hello.save 

DB[:conn].execute("SELECT * FROM songs WHERE name = 'Hello' AND album = '25'")
# => [[1, "Hello", "25"]]

# What happens if we save the same song again?

hello.save

DB[:conn].execute("SELECT * FROM songs WHERE name = 'Hello' AND album = '25'")
# => [[1, "Hello", "25"], [2, "Hello", "25"]]

# Note 
We need our #save method to check to see if the object it is being called on has already been persisted. 
If it is, we shouldn't make it INSERT a new row into the DB 
We should only make it Update the existing one. 

# How Do We Know If An Object Has Been Persisted? 
It will have an id that is not nil 
An object's id attribute gets set only once it has been INSERTed into the DB 

def save 
  if self.id 
    self.update 
  else 
    sql = INSERT INTO SONGS (name, album) VALUES (?, ?)
    DB[:conn].execute(sql, self.name, self.album)
    @id = DB[:conn].execute("SELECT last_insert_rowid() FROM songs")[0][0]
  end 
end 

