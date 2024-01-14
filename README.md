# using-databases-with-render

#Creating a PostgreSQL Instance
To create the PostgreSQL instance where you will store your apps' databases, go to the Render dashboardLinks to an external site., click the "New +" button and select "PostgreSQL".

In the "Name" field, enter a "user-friendly" name for your database. If you only plan to deploy one app to Render, you can provide an appropriate descriptive name (e.g., my_awesome_app_db). Otherwise, you'll probably want to use a more general name (e.g., my_postgres_instance).

Note: Hyphens may not be used in the database names; you should use underscores instead (e.g., my_db, not my-db).

Render will randomly generate identifiers for the "Database" and "User" fields, or you can create those yourself if you prefer.

For "Region", you can either select the location closest to you or you can use the default selection.

For "Region", you can either select the location closest to you or you can use the default selection.

For "PostgreSQL Version", first you need to check which version you have on your local machine. Run psql --version anywhere in your terminal.: psql --version

# Using PSQL to execute database commands
Render makes it pretty easy to accomplish certain database tasks, but there are a number of additional helpful tasks that can't be done through the browser interface. For these, we'll use the PostgreSQL interactive terminal, psqlLinks to an external site..

PSQL is a very helpful tool that allows us to connect to the remote database from our terminal and run certain "meta-commands"Links to an external site. that take the place of running a SQL query (e.g., list our databases; list the data tables in a database). It also includes some useful "utility" commands (e.g., backup a database; restore a database). Finally, we can run SQL commands directly from PSQL.

To get into PSQL, go to your PostgreSQL instance page in the Render dashboard, scroll down to the "Connections" section, and copy the "PSQL Command". Alternatively, you can click "Connect" in the upper right corner, then click "External Connection" and copy the command from there.

Go to your terminal, paste in the command and press enter. 

# Listing Databases
To list the databases on your PostgreSQL instance, run the \l meta-command.
\l

# Listing Data Tables
\dt

# Switching to a Different Database (connect)
\c db_name

# Quitting PSQL
\q

# Creating Multiple Databases within a PostgreSQL Instance
To create a database for a particular app, execute the PSQL Command to open the interactive terminal (if it isn't open already), then run the CREATE DATABASE SQL command:
CREATE DATABASE database_name;

# Backing Up and Recreating Your Databases on Render
with Render's free tier, your PostgreSQL instance and any additional databases you've created on it will expire after 90 days. This means that, before the end of the 90 days, you will need to back up your databases, delete the PostgreSQL instance from Render, create a new PostgreSQL instance, and populate it from the database backups. Render should send an email warning you that your database will be expiring soon.

Before we launch into the instructions for backing up and recreating your Render PostgreSQL instance, let's take a closer look at the PSQL connection string. Go ahead and copy the "PSQL Command" from the Render dashboard then paste it into a new file in your text editor.

Warning: Be careful not to store any of the PSQL commands inside a project repo. Those commands contain secure information so you don't want them to be deployed to GitHub accidentally!

PGPASSWORD=############# psql -h ################-postgres.render.com -U my_db_instance_user my_db_instance

The first element is the password for your database, which will be a 32-character string. Next is the command you're running, in this case, psql. The next component is the host (indicated by the -h flag), which will end with "-postgres.render.com". Next is the name of the database user (indicated by the -U flag), followed, finally, by the name of the database itself.

To create a backup, we're going to modify the PSQL connection string to run PSQL's pg_dumpLinks to an external site. utility command. We'll do that once for each of the two databases we need to back up, starting with bird_app_db. We recommend making the edits to the string in your text editor then copy/pasting it into the terminal when you're done. (But remember not to push it up to Github!)

The first part of the string, the password, will remain the same. The psql command should be updated to pg_dump instead. The host and username should also stay the same. After that, we'll add the following options: 
--format=custom --no-acl --no-owner

The final component of the original connection string is the name of the PostgreSQL instance, my_db_instance. We'll replace that name with bird_app_db instead. After that, we'll add a > to indicate that we want the results of the command to be written to a file, followed by the name we want to use for the backup file, with the .sql extension:

bird_app_db > bird_app_db.sql

The updated string will look something like this: 
PGPASSWORD=############# pg_dump -h ################-postgres.render.com -U my_db_instance_user --format=custom --no-acl --no-owner bird_app_db > bird_app_db.sql

Now that we've created the command, we'll copy/paste it into the terminal (not PSQL) and run it. Make sure you're not inside a directory that's a git repo first to ensure the backup file doesn't get pushed to GitHub. The command will not print any output, but if we run ls, we'll see the newly-created .sql file in the current directory.

To back up our second database, recipe_app_db, all we need to do is replace the name of the database and the name of the backup file and run the modified command:
PGPASSWORD=############# pg_dump -h ################-postgres.render.com -U my_db_instance_user --format=custom --no-acl --no-owner recipe_app_db > recipe_app_db.sql

Now that we've backed up our databases, the next step is to delete the current PostgreSQL instance and create a new one.

# Replacing the Expiring PostgreSQL Instance
To delete the PostgreSQL instance, go to the database page on the Render dashboard. Scroll to the bottom of the page, then click "Delete Database" and follow the instructions.

Next, we'll create a new PostgreSQL instance by clicking the "New +" button and selecting PostgreSQL. Provide a name for the new instance (e.g., my_new_db_instance), then scroll down to the bottom of the page, and click "Create Database."

# Restoring the Databases to the New Instance
Once the new instance has been created, the next step is to create the app-specific databases within that instance.

First we need to execute the PSQL connection string for the new PostgreSQL instance to launch the interactive terminal. Next, we'll run the CREATE DATABASE commands for each of the databases (we're using the same database names but you can use different names if you prefer):
CREATE DATABASE bird_app_db;
CREATE DATABASE recipe_app_db;

Once that's done, we can exit PSQL with the \q command.

Now we're ready to work on building the pg_restoreLinks to an external site. command. Once again, we recommend pasting the connection string into your text editor and editing it there.

The command will consist of the following:

The database password.
The pg_restore command.
The host.
The user.
The options: --verbose --clean --no-acl --no-owner.
The -d flag (for dbname) followed by the name of the new database you're restoring the data to.
The name of the .sql file you're restoring from.
The final string for the bird app will look something like this:

PGPASSWORD=################ pg_restore -h #################-postgres.render.com -U my_new_db_instance_user --verbose --clean --no-acl --no-owner -d bird_app_db bird_app_db.sql

When we run the command in the terminal, we'll see a flurry of activity as it creates the database tables.

Once that's done, we can update the command to use it with the recipe app and run that as well.

# Connecting the New Databases to Your Web Services
The final step in the process is to update each app's Web Service so that it points to the newly-restored database.

From the Render dashboard, we'll select the bird app, then click "Environment" in the nav on the left. Next, we delete the value associated with the DATABASE_URL key and replace it with the Internal URL for the new instance. Remember that we want to connect to the bird app database, not the instance itself, so you need to remove the name of the PostgreSQL instance from the end of the URL and replace it with the name of the bird app database.

After we've saved the change, when we click the app's URL, we should see the JSON for the list of birds.

Finally, we repeat the process for the recipe app.

# Resetting an Existing Database
If you need to reset a database for some reason — say you forgot to comment out the seed data before redeploying an app and you have duplicate data — all you need to do is drop the database and recreate it.

There are two cases to consider. The first is if the database you need to reset is the PostgreSQL instance itself (i.e., if you only have a single app deployed). In this case, you would simply delete and recreate the instance in Render, then connect the new instance to the web service for the app and redeploy it.

If, on the other hand, you need to reset a database within your PostgreSQL instance and not the instance itself, you can do that using PSQL:

Execute the PSQL command for your PostgreSQL instance in the terminal.
Run the SQL command to drop the database: DROP DATABASE database_name;. You should see 'DROP DATABASE' echoed in the terminal. If you don't, make sure you included the semicolon and that your PSQL connection hasn't timed out.
Run the SQL command to create the new database: CREATE DATABASE database_name;
In Render, connect the web service for the app to the new database and redeploy. If you used the same name for the database, you'll just need to redeploy.
Optionally, with either approach, you can also re-seed your database if you choose.

That's it!
