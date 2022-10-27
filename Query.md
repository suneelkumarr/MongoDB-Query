Instructions
Iteration 1

First, we need to import the database we will be using for the lab. We will use the Crunchbase database. Crunchbase is the premier destination for discovering industry trends, investments, and news about hundreds of thousands of companies globally. From startups to Fortune 500s, Crunchbase is recognized as the primary source of company intelligence by millions of users globally.

The database contains more than 18k documents. Each document holds the data about each of the companies. A document looks like the following:

image

    You will find the .zip file of the database on the lab folder.
    Unzip the file
    Navigate to this lab's folder in your terminal, and when inside, import the database to Mongo using the following command:

When running the mongoimport you should be located in the same folder as the data.json file.

$ mongoimport --db companiesDB --collection companies --file data.json

What this mongoimport will do for us is to create a database named companiesDB, and inside the database will create a collection named companies which will be fed with data.json.

Side note: In case errors or hanging with no response when running this command, add --jsonArray at the end of the previous command.

    Check on Mongo Compass if everything goes ok:

image