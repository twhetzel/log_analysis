# Log Analysis

## Description
Log Analysis project for the Udacity Full Stack Nanodegree. The project consists of answering three questions given a database and data.

The data is for a fictional news website. The database is a PostgreSQL database and the schema contains three tables: authors, articles, and log.

## Prerequisites
- Python 2
- PostgreSQL
- psycopg2
- [Vagrant](https://www.vagrantup.com/)
- [VirtualBox](https://www.virtualbox.org/wiki/Downloads)

## Installing
To create an environment to run the project, install Vagrant and Virtual Box.

Start the Virtual Machine(VM) by running `vagrant up` in the directory that contains the "Vagrantfile" and then SSH into the VM using `vagrant ssh`. The first time `vagrant up` is run the VM will be configured based on information in the "Vagrantfile". Navigate to `/vagrant` to access the files shared between the VM and your computer.

Create the database schema and load the data by running `psql -d news -f newsdata.sql`. The "newsdata.sql" file contains the SQL commands to create the schema and load data into the tables.

Note: When done with the VM, type exit and then `vagrant suspend` to pause the VM and maintain the state of the VM.

## Running the Program
To run the program, first create the views by running `psql -d news -f create_views.sql`. For completeness, below are the VIEW statements.

- Create View of Popular Articles
```
CREATE OR REPLACE VIEW favorite_articles as \
    SELECT distinct(count(log.path)) as page_views, log.path \
    FROM log \
    WHERE log.status = '200 OK' \
    and log.path like '/article/%' \
    GROUP BY log.path \
    ORDER BY page_views desc
```

- Create a view of total calls per day.
```
CREATE OR REPLACE VIEW total_calls_per_day as
    SELECT COUNT(*) as all_calls, date_trunc('day', log.time) as date
    FROM log
    GROUP BY date;
```

- Create a view of failed calls per day.
```
CREATE OR REPLACE VIEW failed_calls_per_day as
    SELECT COUNT(*) as failed_calls, date_trunc('day', log.time) as date
    FROM log
    WHERE status != '200 OK'
    GROUP BY date;
```

- Create a view of the percentage of failed calls per day.
```
CREATE OR REPLACE VIEW percent_failed_calls as
    SELECT round((f.failed_calls/t.all_calls*1.0)*100, 1)
    as per_failed_calls, f.date
    FROM failed_calls_per_day as f, total_calls_per_day as t
    WHERE f.date=t.date;
```


Then execute the program as `python analyze_logs.py`. The output is written to the Terminal.

## Example Output
```
Top three favorite articles:
"Candidate is jerk, alleges rival" --- 338647
"Bears love berries, alleges bear" --- 253801
"Bad things gone, say good people" --- 170098

Most popular article authors:
"Ursula La Multa" --- 507594 views
"Rudolf von Treppenwitz" --- 423457 views
"Anonymous Contributor" --- 170098 views
"Markoff Chaney" --- 84557 views

Days where more than 1% of requests lead to errors:
"July 17, 2016" --- 2.3% error
```
