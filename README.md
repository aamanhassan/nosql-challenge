# nosql-challenge
module 12 analysis

Database and Jupyter Notebook Set Up
Use NoSQL_setup_starter.ipynb for this section of the challenge.

Import the data provided in the establishments.json file from your Terminal. Name the database uk_food and the collection establishments. Copy the text you used to import your data from your Terminal to a markdown cell in your notebook.
Within your notebook, import the libraries you need: PyMongo and Pretty Print (pprint).

# Import dependencies
from pymongo import MongoClient
from pprint import pprint

Create an instance of the Mongo Client.
# Create an instance of MongoClient
mongo = MongoClient(port=27017)

Confirm that you created the database and loaded the data properly: List the databases you have in MongoDB. Confirm that uk_food is listed.
# confirm that our new database was created
mongo.list_database_names()
db = mongo['uk_food']

List the collection(s) in the database to ensure that establishments is there.
db.list_collection_names()

Find and display one document in the establishments collection using find_one and display with pprint.
# review the collections in our new database
print(db['establishments'])
# review a document in the establishments collection
pprint(establishments.find_one())

Assign the establishments collection to a variable to prepare the collection for use.
# assign the collection to a variable
establishments = db['establishments']

Use NoSQL_setup_starter.ipynb for this section of the challenge.

The magazine editors have some requested modifications for the database before you can perform any queries or analysis for them. Make the following changes to the establishments collection:

An exciting new halal restaurant just opened in Greenwich, but hasn't been rated yet. The magazine has asked you to include it in your analysis. Add the following information to the database:

post= {
    "BusinessName":"Penang Flavours",
    "BusinessType":"Restaurant/Cafe/Canteen",
    "BusinessTypeID":"",
    "AddressLine1":"Penang Flavours",
    "AddressLine2":"146A Plumstead Rd",
    "AddressLine3":"London",
    "AddressLine4":"",
    "PostCode":"SE18 7DY",
    "Phone":"",
    "LocalAuthorityCode":"511",
    "LocalAuthorityName":"Greenwich",
    "LocalAuthorityWebSite":"http://www.royalgreenwich.gov.uk",
    "LocalAuthorityEmailAddress":"health@royalgreenwich.gov.uk",
    "scores":{
        "Hygiene":"",
        "Structural":"",
        "ConfidenceInManagement":""
    },
    "SchemeType":"FHRS",
    "geocode":{
        "longitude":"0.08384000",
        "latitude":"51.49014200"
    },
    "RightToReply":"",
    "Distance":4623.9723280747176,
    "NewRatingPending":True
}

Find the BusinessTypeID for "Restaurant/Cafe/Canteen" and return only the BusinessTypeID and BusinessType fields.
# Insert the new restaurant into the collection
establishments.insert_one(post)

# Check that the new restaurant was inserted
pprint(establishments.find_one({"BusinessName":"Penang Flavours"}))

Update the new restaurant with the BusinessTypeID you found.

# Find the BusinessTypeID for "Restaurant/Cafe/Canteen" and return only the BusinessTypeID and BusinessType fields
query= {"BusinessType" : "Restaurant/Cafe/Canteen"}
fields= {"BusinessTypeID":1,"BusinessType": 1}
results=establishments.find_one(query,fields)
pprint(results)
# Update the new restaurant with the correct BusinessTypeID
establishments.update_one({'BusinessName': 'Penang Flavours'},{'$set': {'BusinessTypeID': 1}})

# Confirm that the new restaurant was updated
pprint(establishments.find_one({"BusinessName":"Penang Flavours"}))

The magazine is not interested in any establishments in Dover, so check how many documents contain the Dover Local Authority. Then, remove any establishments within the Dover Local Authority from the database, and check the number of documents to ensure they were deleted.

query= {"LocalAuthorityName":"Dover"}
dover_documents= establishments.count_documents(query)
dover_documents
# Delete all documents where LocalAuthorityName is "Dover"
query= {"LocalAuthorityName":"Dover"}
establishments.delete_many(query)
# Check if any remaining documents include Dover
query= {"LocalAuthorityName":"Dover"}
dover_documents= establishments.count_documents(query)
dover_documents
# Check that other documents remain with 'find_one'
pprint(establishments.find_one())


Some of the number values are stored as strings, when they should be stored as numbers.
# Change the data type from String to Decimal for longitude and latitude
establishments.update_many({},[{'$set':{'geocode.latitude':{'$toDouble':'$geocode.latitude'},
                                'geocode.longitude':{'$toDouble':'$geocode.longitude'}}}])
establishments.find_one()
# Set non 1-5 Rating Values to Null
non_ratings = ["AwaitingInspection", "Awaiting Inspection", "AwaitingPublication", "Pass", "Exempt"]
establishments.update_many({"RatingValue": {"$in": non_ratings}}, [ {'$set':{ "RatingValue" : None}} ])

Use update_many to convert RatingValue to integer numbers.
# Change the data type from String to Integer for RatingValue
establishments.update_many({},[{'$addFields':{'RatingValue': {'$toInt':'$RatingValue'}}}])

# Check that the coordinates and rating value are now numbers
establishments.find_one()

Part 3: Exploratory Analysis
Eat Safe, Love has specific questions they want you to answer, which will help them find the locations they wish to visit and avoid.

# Import dependencies
from pymongo import MongoClient
from pprint import pprint
import pandas as pd

# Create an instance of MongoClient
mongo = MongoClient(port=27017)

# assign the uk_food database to a variable name
db = mongo['uk_food']

# review the collections in our database
print(db.list_collection_names())

# assign the collection to a variable
establishments = db['establishments']

Use count_documents to display the number of documents contained in the result.Which establishments have a hygiene score equal to 20?

# Find the establishments with a hygiene score of 20
query = {'scores.Hygiene': 20}
results = establishments.find(query)
# Use count_documents to display the number of documents in the result
print(f"No of documents in results : {establishments.count_documents(query)}")

Display the first document in the results using pprint.
# Display the first document in the results using pprint
pprint(results[0])

Convert the result to a Pandas DataFrame, print the number of rows in the DataFrame, and display the first 10 rows.

# Convert the result to a Pandas DataFrame
hygiene_df= pd.DataFrame(results)

# Display the number of rows in the DataFrame
print(f"No of rows in Dataframe: {len(hygiene_df)}")
# Display the first 10 rows of the DataFrame
hygiene_df.head()

Which establishments in London have a RatingValue greater than or equal to 4?
# Find the establishments with London as the Local Authority and has a RatingValue greater than or equal to 4.
query ={'LocalAuthorityName': {'$regex':"London"},'RatingValue':{'$gte': 4}}
results= establishments.find(query)

# Use count_documents to display the number of documents in the result
print(f" Number of documents in the results : {establishments.count_documents(query)}")
# Display the first document in the results using pprint
pprint(results[0])

# Convert the result to a Pandas DataFrame
london_df = pd.DataFrame(results)
# Display the number of rows in the DataFrame
print(f"No of rows in dataframe: {len(london_df)}")
# Display the first 10 rows of the DataFrame
london_df.head(10)

What are the top 5 establishments with a RatingValue of 5, sorted by lowest hygiene score, nearest to the new restaurant added, "Penang Flavours"?
# Search within 0.01 degree on either side of the latitude and longitude.
# Rating value must equal 5
# Sort by hygiene score

degree_search = 0.01
latitude = 51.49014200
longitude = 0.08384000

# Search within 0.01 degree on either side of the latitude and longitude.
# Rating value must equal 5
query = {
        'geocode.latitude':{'$gte':latitude-degree_search,'$lte': latitude+degree_search},
        'geocode.longitude':{'$gte':longitude-degree_search ,'$lte':longitude+degree_search},
        'RatingValue': 5
        } 
# Sort by hygiene score
sort =  [('sort.Hygiene' , 1)]
limit = 5

# Print the results

pprint(list(establishments.find(query).sort(sort).limit(limit)))

# Convert result to Pandas DataFrame
pd.DataFrame(establishments.find(query).sort(sort).limit(limit))

How many establishments in each Local Authority area have a hygiene score of 0? Sort the results from highest to lowest, and print out the top ten local authority areas.

# Create a pipeline that: 
# 1. Matches establishments with a hygiene score of 0
# 2. Groups the matches by Local Authority
# 3. Sorts the matches from highest to lowest
match_query= {'$match':{'scores.Hygiene' : 0}}
group_query= {'$group': {'_id': "$LocalAuthorityName", 'count': { '$sum': 1 }}}
sort_values = {'$sort': { 'count': -1 }}

pipeline= [match_query,group_query,sort_values]

# Print the number of documents in the result
results= list(establishments.aggregate(pipeline))
# Print the first 10 results
pprint(results[0:10])

# Convert the result to a Pandas DataFrame
results_df= pd.DataFrame(results)

# Display the number of rows in the DataFrame
print(f" No of rows in the DataFrame: {len(results_df)}")
# Display the first 10 rows of the DataFrame
results_df.head(10)
