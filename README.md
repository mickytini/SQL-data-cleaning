This is an educational project on data cleaning and preparation using SQL. The original database in CSV format is located in the file club_member_info.csv. Here, we will explore the steps that need to be applied to obtain a cleansed version of the dataset.

Let's inspect the initial rows to analyze the data in its original format.

    SELECT * FROM club_member_info LIMIT 10

The result:
|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|addie lush|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|      ROCK CRADICK|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel Sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|  Gaylor Redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar       |44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann Kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|   Joete Cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
| fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|

# Copy the table

**Create a new table for cleaning**

Let's generate a new table where we can manipulate and restructure the data without modifying the original dataset.

    CREATE TABLE club_member_info_cleaned (
	    full_name VARCHAR(50),
	    age INTEGER,
	    martial_status VARCHAR(50),
	    email VARCHAR(50),
	    phone VARCHAR(50),
	    full_address VARCHAR(50),
	    job_title VARCHAR(50),
	    membership_date VARCHAR(50)
    );

**Copy all values from original table**

    INSERT INTO club_member_info_cleaned
    SELECT * FROM club_member_info;

# Clean data and document it

- Full_name column: Inconsistent letter case
   Remove whitespace characters before and after the string

      UPDATE club_member_info_cleaned
      SET full_name = TRIM(full_name);

   Reformat string

      UPDATE table_name
      SET full_name = UPPER(SUBSTR(full_name, 1, 1)) || LOWER(SUBSTR(full_name, 2))
      WHERE full_name IS NOT NULL;

The result:
|full_name|age|martial_status|email|phone|full_address|job_title|membership_date|
|---------|---|--------------|-----|-----|------------|---------|---------------|
|Addie lush|40|married|alush0@shutterfly.com|254-389-8708|3226 Eastlawn Pass,Temple,Texas|Assistant Professor|7/31/2013|
|Rock cradick|46|married|rcradick1@newsvine.com|910-566-2007|4 Harbort Avenue,Fayetteville,North Carolina|Programmer III|5/27/2018|
|Sydel sharvell|46|divorced|ssharvell2@amazon.co.jp|702-187-8715|4 School Place,Las Vegas,Nevada|Budget/Accounting Analyst I|10/6/2017|
|Constantin de la cruz|35||co3@bloglines.com|402-688-7162|6 Monument Crossing,Omaha,Nebraska|Desktop Support Technician|10/20/2015|
|Gaylor redhole|38|married|gredhole4@japanpost.jp|917-394-6001|88 Cherokee Pass,New York City,New York|Legal Assistant|5/29/2019|
|Wanda del mar|44|single|wkunzel5@slideshare.net|937-467-6942|10864 Buhler Plaza,Hamilton,Ohio|Human Resources Assistant IV|3/24/2015|
|Joann kenealy|41|married|jkenealy6@bloomberg.com|513-726-9885|733 Hagan Parkway,Cincinnati,Ohio|Accountant IV|4/17/2013|
|Joete cudiff|51|divorced|jcudiff7@ycombinator.com|616-617-0965|975 Dwight Plaza,Grand Rapids,Michigan|Research Nurse|11/16/2014|
|Mendie alexandrescu|46|single|malexandrescu8@state.gov|504-918-4753|34 Delladonna Terrace,New Orleans,Louisiana|Systems Administrator III|3/12/1921|
|Fey kloss|52|married|fkloss9@godaddy.com|808-177-0318|8976 Jackson Park,Honolulu,Hawaii|Chemical Engineer|11/5/2014|

- Age column:
   Age out of realistic range

      UPDATE club_member_info_cleaned 
      SET age = SUBSTR(age , 1, 2)
      WHERE LENGTH(age) > 2;

   The data in the cell must only have 2 digits from 0 to 99. If there are 3 or more digits, only the first 2 numbers will be taken. If the cell is empty, replace it with the number 40 (median).

      UPDATE club_member_info_cleaned 
      SET age = CASE
      WHEN LENGTH(age) = 0 OR age IS NULL THEN '40'
      WHEN LENGTH(age) > 2 THEN SUBSTR(age , 1, 2)
      ELSE age
      END;

   If any cell is blank, fill in the number: 40 (median)

      UPDATE club_member_info_cleaned
      SET age = 40
      WHERE age IS NULL OR age = '';

- Martial_status column: If any cell is blank (no data), fill in: "single"

      UPDATE club_member_info_cleaned 
      SET martial_status = 'single'
      WHERE martial_status IS NULL OR martial_status = '';

- Phone column: If any cell is blank (no data), fill in: "NULL"

      UPDATE club_member_info_cleaned 
      SET phone = 'NULL'
      WHERE phone IS NULL OR phone = '';

- Job_title column: If any cell is blank (no data), fill in: "NULL"

      UPDATE club_member_info_cleaned 
      SET job_title = 'NULL'
      WHERE job_title IS NULL OR job_title = '';

- Membership_date column: I found 16 members with join dates of **191x or 192x** - which doesn't seem right for their age. Please help me fix this data if possible :D

# Clean up duplicate data

- Full_name column:
  
      -- Identify the duplicates
      SELECT full_name , COUNT(*)
      FROM club_member_info_cleaned 
      GROUP BY full_name 
      HAVING COUNT(*) > 1;

      -- Delete the duplicates while keeping only one occurrence
      DELETE FROM club_member_info_cleaned
      WHERE ROWID NOT IN (
         SELECT MIN(ROWID)
         FROM club_member_info_cleaned
         GROUP BY full_name
      );

- Email column:

      -- Identify the duplicates
      SELECT email , COUNT(*)
      FROM club_member_info_cleaned 
      GROUP BY email 
      HAVING COUNT(*) > 1;

      -- Delete the duplicates while keeping only one occurrence
      DELETE FROM club_member_info_cleaned
      WHERE ROWID NOT IN (
         SELECT MIN(ROWID)
         FROM club_member_info_cleaned
         GROUP BY email
      );
  
- Phone column:

      -- Identify the duplicates
      SELECT phone , COUNT(*)
      FROM club_member_info_cleaned 
      GROUP BY phone 
      HAVING COUNT(*) > 1;

      -- Delete the duplicates while keeping only one occurrence
      DELETE FROM club_member_info_cleaned
      WHERE ROWID NOT IN (
         SELECT MIN(ROWID)
         FROM club_member_info_cleaned
         GROUP BY phone
      );

- Full_address column:

      -- Identify the duplicates
      SELECT full_address , COUNT(*)
      FROM club_member_info_cleaned 
      GROUP BY phone 
      HAVING COUNT(*) > 1;

      -- Delete the duplicates while keeping only one occurrence
      DELETE FROM club_member_info_cleaned
      WHERE ROWID NOT IN (
         SELECT MIN(ROWID)
         FROM club_member_info_cleaned
         GROUP BY full_address
      );
  
