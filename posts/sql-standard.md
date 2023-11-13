---
layout: default
parent: Post
title: SQL - Data Standards
nav_order: 2
---

## Naming
- Table:
   - Use snake case: lowercase, underscores instead of spaces, no numbers to start a name, and no special characters, no “key words”
      - Good examples:
         - user_status
         - trigger_method
         - release_tag_map
      - Bad examples:
         - UserStatus
         - TRIGGER_METHOD
         - 10release_tag_map
      > Why: if we use the above standards, then we do not need to put double quotes around columns

- Column/Field:
   - Use snake case: lowercase, underscores instead of spaces, no numbers to start a name, and no special characters, no “key words”
   - Primary key can be id.
      - MUST be an id (auto incrementing integers - tiny, small, medium or big ints either way, simple numbers)
   - Foreign key column must have a table name with their primary key
      - Pattern: other_table_id (the `_id` is the pattern to let you know that this is a foreign key to a different table)
      - Examples:
         - user_id => this is a foreign key to the “id” column in “user” table
         - release_tag_id => this is a foreign key to the “id” column in “release_tag” table
   - Avoid using columns with the same name as the table name. This can cause confusion while writing query
   - Avoid abbreviated, concatenated, or acronym-based names
   - Avoid using any word that is considered a reserved word in the database 
   - Always have an id field as the FIRST field (auto incrementing, integer based)
   - Always have a created_on as a LAST field (there are two optional exceptions below)
   - OPTIONAL IF: a table can be updated, ALWAYS have an updated_on field after created_on
   - OPTIONAL IF: a table can soft delete, ALWAYS have an deleted_on field after created_on
   - Example of creation:
      - Create table x ( created_on timestamp default current_timestamp );
      - THESE ARE DATABASE RECORDS.  They can be changed by inserts/updates/deletes/etc.  These should NOT be trusted as official times things happen for users.  Make a new column if you want a dedicated time for that. 
   - Example:
      - Create table x (
      - built_on timestamp,
      - created_on default current_timestamp
				)
In this case, built_on can only ever change if you ACTIVELY change the value in that column.  “Created on” can change just to a handful of different database related activities
Always use the smallest possible sized types.  Example:
https://dev.mysql.com/doc/refman/8.0/en/integer-types.html
Tinyint: 1 byte of storage, int: 4 bytes, bigint: 8 bytes.
1,000,000 rows of a tinyint: 1 meg of space
1,000,000 rows of a tinyint: 4 meg of space
1,000,000 rows of a tinyint: 8 meg of space
Note - this also includes MEMORY.  If you do a “sort” in the database, it has to load all those records into memory to sort.  The bigger the values, the less it can sort at once, and the slower the whole query becomes.
Common Column Names:
“Label” instead of “name” since name is often referring to a persons name… or a title of something … “label” is more generic and a good pattern to use UNLESS you explicitly need to use something else. 
Build table: label
Countries: label
User table: first_name, last_name
Royalty table: person_title
Default Ordering Columns
All columns should be ordered from smallest to biggest. There are specific exceptions to this to make the tables more optimal as well as easier to do quick information gathering on, but the primary way to sort columns (unless there is a very specific reason why to do this otherwise) is to order by size of information in columns. Order should read from left to right:
Boolean
tinyint, smallint, mediumint, int, bigint
float, double
datetime, timestamp
char(Example: 5), char(Example: 30), char(Example: 66) - try to avoid char's unless very specifically needed. Varchars use less space and can be faster as a result
varchar(Example: 5), varchar(Example: 30), varchar(Example: 65)
text, json, xml
Blobs

- Indexes:
   - Indexes should be explicitly named and include both the table name and the column name(s) indexed. Including the column names make it much easier to read through SQL explain plans. If an index is named foobar_ix1 then you would need to look up what columns that index covers to understand if it is being used correctly.
   - Examples:
      - `create unique index tags_alias_idx on tags (alias);`
         - Official name: `tags_alias_idx`
      - `create unique index trigger_method_alias_uidx on trigger_method (alias);`
         - Official name: `trigger_method_alias_idx`
      - `create unique index build_tag_map_build_tag_idx on release_tag_map (user_id, tag_id);`
         - Official name: `release_tag_map_user_tag_idx`
         - NOTE: this didn’t have the release_id_tag_id_idx since i shortened it.  Sometimes this is needed if the index name is too long.  It’s a matter of TRYING to me descriptive that is key
         - Also … “_idx” really isn’t needed, but sometimes it makes it easier to see that it’s an index if you look up information in the information schema (which can mix tables, columns, index, etc in a single column… depending on the query)

- Constraints:
   - Constraints should be given descriptive names. This is especially true for check constraints. It's much easier to diagnose an errant insert if the check constraint that was violated lets you know the cause
   - Don’t use foreign key constraints.  This cause far more problems than they solve

## Data Type
- Boolean
   - Booleans should start with is, has, was, or should (eg. is_created, is_enabled, has_siblings, was_executed, should_clear_cache)
   - In the above examples, the was_executed, should_clear_cache should probably be statuses, but the idea for booleans is still valid
- Timestamp, Date
   - All timestamps and date field names should adhere to the following conventions:
      "at" represents a UNIX timestamp "created_at"
      "on" represents an ISO date "created_on"
      "is" represents a boolean "is_created" (see Booleans)
      {
         "created_at": 1377007762.1234,
         "created_on": "2013-10-20T17:15:57.185Z",
         "is_created": True
      }
   - This pattern should be carried across all relevant terms: "created", "updated", "purchased", "started", "queued", "finished", "converged"
   - All stored objects should contain created_at, updated_at and (when using soft deletes) deleted_at timestamps.
   - Additional options for specific standalone “date” or “time” fields
      - Date example: “finalized_date” (2023-10-10) – only a date, no time
      - Time example: “finalized_time” (22:56:13) – only a time, no date
- Null and Default Values
   - Only allow nulls in columns when necessary. Usually, nulls are only allowed in the following situations:
   - A Foreign Key column where the relationship is optional
   - A Date, DateTime or DateTimeOffset column where a value may be missing.
   - Reasons or this is that we often want to create DEFAULT fields instead.  This is often debated by database administrators DBA’s
      - Example:
         - Create table user ( age tinyint default 0 )
            - In this case, if they don’t give an age… use 0
            - Select * from users where age != 0
         - Create table user ( age tinyint )
            - If we have 0 as a default value in some things … then you would have to use 2 where clauses
            - Select * from users where age != 0 and age is not null

## Deleting vs. Hiding Rows
- In almost all situations, favor hiding rows by setting a flag in the row instead of deleting. Create a bit column is_hidden or is_inactive with a default value of false (0) or deteled_on with the current date of deletion.
- DEFAULT: always “soft delete” (aka: hiding) for anything that isn’t a “_map” table.  
- When foreign key relationships are used in a database, deleting rows can become quite difficult. In addition, there is no implicit audit trail in SQL Server when rows are deleted. If something shows up on a report one day and is just missing the next with no explanation, that can be a maintenance nightmare.
- I did not find any utility tools to facilitate and enforce adherence to data standards
