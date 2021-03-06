# Image Vote
Experimental Node.js server app exposing a RESTful API for uploading own photos and voting for other users´ photos.

## Prerequisites
Install Node.js
  see http://nodejs.org/

Install MongoDB
  see https://www.mongodb.org/

Install project dependencies:

`$ npm install`

Run database:

`$ mongod`

Run server (Default port is 3000):

`$ node app.js`

## REST API
All resources have `_id` as ID attribute.

### Register User
The application enforces user registration. All subsequent API calls require authentication using HTTP Basic authentication.

#### URL
`POST /api/users`

#### Data params
`{email: [String (required)], password: [String (required)], gender: [enum: ['male', 'female'] (required)]}`

#### Success response
`201, {_id: [String], email: [String], gender: [enum: ['male', 'female']}`

#### Error response
`409, {message: 'Email already registered'}`

#### Sample call
`$ curl -d '{"email":"hello@example.org","password":"qwerty","gender":"male"}' -i -H "Accept: application/json" -H "Content-Type: application/json" -X POST http://127.0.0.1:3000/api/users`

### Create and upload Photo
Users may create and upload photos. An array of created photo IDs are provided when later creating a contact sheet.

#### URL
`POST /api/photos`

#### Data params
`{photo: [binary image file data]}`

#### Success response
`201, {_id: [String], path: [String], userId: [String], displayCount: [Number], likedBy: [Array(String)]}`

#### Error responses
`404, {message: 'User not found'}`

`400, {message: 'Missing photo'}`

#### Sample call
`$ curl -F photo="@/local/path/to/photo.jpg" -i -H "Accept applciation/json" -X POST http://hello%40example.org:qwerty@127.0.0.1:3000/api/photos`

### Create Contact Sheet
The contact sheet holds a collection of photos, and keeps track of how many other photos the user has voted for (liked).

#### URL#
`POST /api/sheets`

#### Data params
`{photoIds: [Array(String)]}`

#### Success response
`201, {_id: [String], userId: [String], voteCount: [Number], photos: [Array(Photo)]}`

#### Error responses
`404, {message: 'User not found'}`

`400, {message: 'Missing photos'}`

`400, {message: 'Minimum of X photos must be provided'}`

#### Sample call
`$ curl -d '{"photoIds":["5352a2133fad13b40bdd64b1","5352bf963fad13b40bdd64b2"]}' -i -H "Accept: application/json" -H "Content-Type: application/json" -X POST http://hello%40example.org:qwerty@127.0.0.1:3000/api/sheets`

### Get Photos
The user can access a list of random photos of other users. Two photos are returned by default.

#### URL
`GET /api/photos?count=4`

#### Success response
`200, [{_id: [String], path: [String], userId: [String], displayCount: [Number], likedBy: [Array(String]}, ...]`

#### Sample call
`$ curl -i -H "Accept: application/json" http://hello%40example.org:qwerty@127.0.0.1:3000/api/photos`

### Like Photo
In the context of a Contact Sheet the user can like other users´ photos. The response indicates how many photos the user needs to like before getting access to likes for own photos.

#### URL
`POST /api/sheets/:id/likes`

#### Data params
`{photoId: [String]}`

#### Success response
`200, {votesLeft: [Number]}`

#### Error responses
`404, {message: 'User not found'}`

`404, {message: 'Sheet not found'}`

`403, {message: 'Permission to sheet denied'`

`404, {message: 'Photo not found'}`

`400, {message: 'Photo already liked by user'}`

#### Sample call
`$ curl -d '{"_id":"5350eb54f9d135080453428b"}' -i -H "Accept: application/json" -H "Content-Type: application/json" -X POST http://hello%40example.org:qwerty@127.0.0.1:3000/api/sheets/5352c844463671180ee586ae/likes`

### Access likes for own photos
Returns results for photos in own contact sheet or empty content if user has not liked enough of other users´ photos.

#### URL
`GET /api/sheets/:id/likes`

#### Success responses
`304`

`200, [{_id: [Number], path: [String], userId: [String], displayCount: [Number], likedBy: [Array(String]}]`

#### Error responses
`404, {message: 'User not found'}`

`404, {message: 'Sheet not found'}`

`403, {message: 'Permission denied'}`

#### Sample call
`curl -i -H "Accept: application/json" http://hello%40example.org:qwerty@127.0.0.1:3000/api/sheets/5352c844463671180ee586ae/likes`

## Client considerations
The client should store user credentials needed to authenticate. The client should also store the ID for its current contact sheet.



