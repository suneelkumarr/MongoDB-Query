Answers
1. All the companies whose name match 'Babelgum'. Retrieve only their name field.

In Mongo Compass:

    Query: { name: "Babelgum" }
    Projection: { name: 1, _id: 0 }

In Mongo Shell:

db.companies.find({ name: "Babelgum" }, { name: 1, _id: 0 })


2. All the companies that have more than 5000 employees. Limit the search to 20 companies and sort them by number of employees.

In Mongo Compass:

    Query: { number_of_employees: { $gt: 5000 } }
    Sort: { number_of_employees: -1 }
    Limit: 20

In Mongo Shell:

db.companies.find({ number_of_employees: { $gt:5000 } }).sort({ number_of_employees: -1 }).limit(20)


3. All the companies founded between 2000 and 2005, both years included. Retrieve only the name and founded_year fields.

In Mongo Compass:

    Query: { $and: [{founded_year: {$gte: 2000}}, {founded_year:{$lte: 2005}}] }
    Projection: {name: 1, founded_year: 1, _id: 0}

In Mongo Shell:

db.companies.find(
  { $and: [{founded_year: {$gte: 2000}}, {founded_year:{$lte: 2005}}] }, 
  { name: 1, founded_year: 1, _id: 0 }
)


4. All the companies that had a Valuation Amount of more than 100.000.000 and have been founded before 2010. Retrieve only the name and ipo fields.

In Mongo Compass:

    Query: { $and: [{"ipo.valuation_amount": {$gt: 10000000}}, {founded_year:{$lt: 2010}}] }
    Projection: { name: 1, founded_year: 1, ipo: 1, _id: 0 }

In Mongo Shell:

db.companies.find(
  { $and: [{"ipo.valuation_amount": {$gt: 10000000}}, {founded_year:{$lt: 2010}}] }, 
  {name: 1, founded_year: 1, ipo: 1, _id: 0}
)


5. All the companies that have less than 1000 employees and have been founded before 2005. Order them by the number of employees and limit the search to 10 companies.

In Mongo Compass:

    Query: { $and: [{ number_of_employees: {$lt: 1000} }, { founded_year: {$lt: 2005} }] }
    Sort: { number_of_employees: -1 }
    Limit: 10

In Mongo Shell:

db.companies.find({ $and: [{ number_of_employees: {$lt:1000} }, { founded_year: {$lt: 2005} }] }).sort({ number_of_employees: -1 }).limit(10)


6. All the companies that don't include the partners field.

In Mongo Compass:

    Query: { partners: { $exists: false } }

To get the companies that don't have partners, you can do:

    Query: { partners: { $size: 0 } }

In Mongo Shell:

db.companies.find({ partners: { $exists: false } })

or

db.companies.find({ partners: { $size: 0 } })


7. All the companies that have a null type of value on the category_code field.

In Mongo Compass:

    Query: { category_code: { $type: "null" } }

In Mongo Shell:

db.companies.find({ category_code: { $type: "null" } })


8. All the companies that have at least 100 employees but less than 1000. Retrieve only the name and number of employees fields.

In Mongo Compass:

    Query: { $and: [{ number_of_employees: {$gte: 100} }, { number_of_employees: {$lt: 1000} }] }
    Projection: {name: 1, number_of_employees: 1, _id: 0}

In Mongo Shell:

db.companies.find(
  { $and: [{number_of_employees: {$gte: 100}}, {number_of_employees: {$lt: 1000}}]}, 
  {name: 1, number_of_employees: 1, _id: 0}
)


9. Order all the companies by their IPO price in a descending order.

In Mongo Compass:

    Sort: { "ipo.valuation_amount": -1 }

In Mongo Shell:

db.companies.find().sort({ "ipo.valuation_amount": -1 })

(You will need to specify a limit to avoid the executor error)

10. Retrieve the 10 companies with more employees, order by the number of employees

In Mongo Compass:

    Sort: { number_of_employees: -1 }
    Limit: 10

In Mongo Shell:

db.companies.find().sort({ number_of_employees: -1}).limit(10)


11. All the companies founded on the second semester of the year. Limit your search to 1000 companies.

In Mongo Compass:

    Query: { founded_month: { $gte: 7 } }
    Limit: 1000

In Mongo Shell:

db.companies.find({ founded_month: { $gte: 7 } }).limit(1000)


12. All the companies founded before 2000 that have an acquisition amount of more than 10.000.000

In Mongo Compass:

    Query: { $and: [{ founded_year: {$lt: 2000} }, {"acquisition.price_amount": { $gt: 10000000 }}] }

In Mongo Shell:

db.companies.find({ $and: [{ founded_year: {$lt: 2000} }, { "acquisition.price_amount": { $gt: 10000000 } }] })


13. All the companies that have been acquired after 2010, order by the acquisition amount, and retrieve only their name and acquisition field.

In Mongo Compass:

    Query: { "acquisition.acquired_year": { $gt: 2010 } }
    Projection: { name: 1, acquisition: 1, _id: 0 }
    Sort: { "acquisition.price_amount": -1 }

In Mongo Shell:

db.companies.find({ "acquisition.acquired_year": { $gt: 2010 } }, { name: 1, acquisition: 1, _id: 0 }).sort({ "acquisition.price_amount": -1 })


14. Order the companies by their founded year, retrieving only their name and founded year.

In Mongo Compass:

    Projection: { name: 1, founded_year: 1, _id: 0 }
    Sort: { founded_year: -1 }

In Mongo Shell:

db.companies.find({}, { name: 1, founded_year: 1, _id: 0 }).sort({ founded_year: -1 })

(You will need to specify a limit to avoid the executor error)

15. All the companies that have been founded on the first seven days of the month, including the seventh. Sort them by their acquisition price in a descending order. Limit the search to 10 documents.

In Mongo Compass:

    Query: { founded_day: { $lte: 7 } }
    Sort: { "acquisition.price_amount": -1 }
    Limit: 10

In Mongo Shell:

db.companies.find({ founded_day: { $lte: 7 } }).sort({ "acquisition.price_amount": -1 }).limit(10)


16. All the companies on the 'web' category that have more than 4000 employees. Sort them by the amount of employees in ascending order.

In Mongo Compass:

    Query: { $and: [{ category_code: "web" }, { number_of_employees: {$gt: 4000} }] }
    Sort: { number_of_employees: 1 }

In Mongo Shell:

db.companies.find({ $and: [{category_code: "web"}, { number_of_employees: {$gt: 4000} }] }).sort({ number_of_employees: 1 })


17. All the companies whose acquisition amount is more than 10.000.000, and currency is 'EUR'.

In Mongo Compass:

    Query: { $and: [{ "acquisition.price_amount": {$gt: 10000000} }, { "acquisition.price_currency_code": "EUR" }] }

In Mongo Shell:

db.companies.find(
  { $and: [{ "acquisition.price_amount": {$gt: 10000000} }, 
  { "acquisition.price_currency_code": "EUR" }] }
)  


18. All the companies that have been acquired on the first trimester of the year. Limit the search to 10 companies, and retrieve only their name and acquisition fields.

In Mongo Compass:

    Query: { "acquisition.acquired_month": { $lt: 4 } }
    Projection: {name: 1, acquisition: 1, _id: 0}
    Limit: 10

In Mongo Shell:

db.companies.find({ "acquisition.acquired_month": { $lte: 3} }, { name: 1, acquisition: 1, _id: 0 }).limit(10)


19. All the companies that have been founded between 2000 and 2010, but have not been acquired before 2011.

In Mongo Compass:

    Query: { $and: [{ founded_year: {$gte: 2000} }, { founded_year: {$lte: 2010} }, { "acquisition.acquired_year": {$gte: 2011} }] }

In Mongo Shell:

db.companies.find( 
  { $and: [{ founded_year: {$gte: 2000} }, { founded_year: {$lte: 2010} }, { "acquisition.acquired_year": {$gte: 2011} }] }
)