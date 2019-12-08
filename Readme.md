## Flow

- When a vehicle enters, it sensor sends a post request to the /entry api of the app, which in turn published the entry record to the raw topic of the kafka cluster. The request body has the following structure
```javascript
{
  "_id": "20343434995051",
  "vehicle_id": "23554dfd77hh343"
	"entry_timestamp": 1575768689,
	"parking_loc_id": "239499fdf939494",
	"location": "sdsdsdsd"
}
```
- Meanwhile, the consumer-producer, which is subscribed to the raw topic, will get these messages as an when it is published. It then goes on to process the data by hooking up with various other data sources and publishes the processed data to MongoDB. Currently, the proccessed data has the following structure
```javascript
{
  "_id": "20343434995051",
  "vehicle_id": "23554dfd77hh343"
	"entry_timestamp": 1575768689,
	"parking_loc_id": "239499fdf939494",
  "location": "sdsdsdsd",
  "is_wanted": False,
  "rate_per_hour": 2,   //rate per hour is dynamic based on time of the day and location
  "accumulated_penalty": 50
}
```
- When the vehicle leaves, the sensor sends a post request to the /exit endpoint of the app, which then fetches the processed record of that vehicle and calcuates the total amount due based on the exit timestamp and the accumulated penalty. A transaction wil be then attempted on the blockchain and is handled accordingly. The post body request has the following structure
```javascript
{
  "_id": "20343434995051",
  "vehicle_id": "23554dfd77hh343"
	"exit_timestamp": 1575794626,
	"parking_loc_id": "239499fdf939494",
	"location": "sdsdsdsd"
}
```

## Setup

Pre-req
- Kafka
- MongoDB
- Anaconda (recommended)

1. Git clone
2. cd to directory
3. conda env create -f environment.yml
4. conda activate realtime-app (exact command might not work, if it doesn't change accordingly)
5. create .env and set the config
6. Run flask app in one terminal
   - python src/app.py or
   - FLASK_APP=src/app.py FLASK_ENV=development flask run --port 3500 (for hot reloading)
7. Run consumer-producer.py in another terminal
   - python src/consumer-producer.py
8. Send entry and exit requests from Postman. Update the _id parameter of the entry request body to avoid having issues with mongodb. Result of the exit request will show the total parking amount.