
Some projects built with AWS services
Newsbreak Clone
```
A news app that can recommend summarized news to users based on their view history 
```
![alt text](https://github.com/duochen13/Backend-Architecture-Design/blob/master/pictures/newsbreak_clone.jpg?raw=true)

Photo Thumbnail
```
An application that can enable users to view/store tagged pictures
```
![alt text](https://github.com/duochen13/Backend-Architecture-Design/blob/master/pictures/photo_thumbnail.png?raw=true)

API and Lambda Functions
```
\upload:
Trigger LB0, LB1
Description: User upload original photo, first send to SQS through LB0, and LB1 helps extract photos from SQS queue, and do “image processing”. The generated thumbnail and taggings was stored in ES and DynamoDB. And the actual image file (the original image) file is stored in S3 bucket
```
```
\search
Trigger LB3
Description: User types some keywords (taggings in this case). LB3 connects with ES, given the tag, ES will return a list of images (with ids) of the user input tag. 
```
```
\imgid=<img_id>
Trigger LB4
Description: When user tap the thumbnail, LB4 helps find the corresponding image id of the thumbnail through ES and extract original image file in the S3 bucket based on image id. And return the original image through LB6 (should be LB6 not LB3 as marked on photo)
```

Real-time Restaurant Route Status
```
A real-time application that can inform users how busy the restaurant is and how busy the restaurant is typically,
and route status such as congestion and accidents.
```
![alt text](https://github.com/duochen13/Backend-Architecture-Design/blob/master/pictures/event_driven_restaurant_route_status.png?raw=true)

API and Lambda Functions: 
```
1.GET /search?restaurant=<>      LB1 (Restaurant Status Microservice)
LB1 will be triggered. LB1 will answer the question “How busy is it typically at other hours?”, through querying in ElasticSearch(ES1), which will return satisfied restaurant ids, and then we query in DynamoDB restaurant table based on ids, it will return a map where the key is the time slot, and the corresponding key is the ‘busy level’ (quantified values, can be the value such as the approximate number of people occupied at current time). 
LB1 will also answer the question “How busy the place is right now” by sending event stream data to Kafak in the format of ‘Date’: “2021-04-20”,  Time: “14:02pm”. userId: XXX, processed by Kafka streaming services.
```
```
2. GET /search?destination=<>      LB2 (Route Status Microservice)
Based on the destination query and current location query, LB2 interacts with Google Map API to get the latest information about the route status (congestion or accidents). And send the data(route information) to SQS queue.
```
```
LB3 (Consumer, annotate ‘busy level’ of restaurants)
Based on the stream event data sent to Kafka, we group event data by time slot, generate keys using time intervals such as ‘14pm-16pm’, ‘16pm-18pm’, for example, event data ‘“2021-04-20”,  Time: “14:02pm”. userId: XXX’ sent from LB1 will be mapped to “14pm-16 pm” time interval, we then count the frequency of each time interval key to create a “busy level” table of each restaurant. Then we store the restaurant id and corresponding “busy level” table to elasticsearch for LB1 to query in the future.
```
```
LB5 (Extract messages from SQS queue, sorry no LB4, typo error)
In order to send real-time notifications regarding the route status (accident or congestion) to users, we use SQS queue as a buffer to store the messages, and use SNS services to notify users, LB5 simply serves the role as extracting messages from SQS queue
```
The backend architecture diagrams from COMS6998 Cloud Computing with Big Data at Columbia

