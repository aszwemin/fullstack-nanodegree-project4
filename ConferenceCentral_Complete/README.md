Conference App for project 4 of fullstack nanodegree.

## How to run the project

1. Go to `project5-123.appspot.com/_ah/api/explorer`
2. You can use the api explorer to test the application
3. For task 1, the following endpoints are available:
  * `getConferenceSessions(websafeConferenceKey)`
  * `getConferenceSessionsByType(websafeConferenceKey, typeOfSession)`
  * `getSessionsBySpeaker(speaker)`
  * `createSession(SessionForm, websafeConferenceKey)`
4. For task 2, the following endpoints are available:
  * `addSessionToWishlist(SessionId, websafeConferenceKey)`
  * `getSessionsInWishlist()`
5. For task 3, the following endpoints are available:
  * `getSessionsByLocation(location)`
  * `getSessionsByDate(date)`
6. For task 4, the following endpoint is available:
  * `getFeaturedSpeaker()`

## Task 1 design decisions

### Session

The session class has following fields:
* `name = ndb.StringProperty(required=True)` - a string property representing the name of the session
* `highlights = ndb.TextProperty()` - a text property (as highlights can be a longer text) representing the highlights of the session
* `speaker = ndb.StringProperty()` - a string property (decided on having just a string property instead of a separate entity) representing the speaker of the session
* `date = ndb.DateProperty()` - a date property (converted in code from user input) representing the date of the session
* `start_time = ndb.TimeProperty()` - a time property (converted in code from user input) representing the time of the session 
* `duration_mins = ndb.IntegerProperty()` - an integer property (as it's a number with no need for decimal places) representing the duration of the session in minutes
* `type_of_session = ndb.StringProperty()` - a string property (limited to the values of SessionType) representing the type of the session
* `location = ndb.StringProperty()` - a string property (didn't want to use the location property as this is more the name of the room) representing the location of the session

### Speaker

Speaker was not implemented as a separate entity, instead it's just a string property on the session. This decission was made mainly due to available time, as if I would add speaker as a separate entity, Iwould also have to implement api endpoints for getting and setting the speaker and with my current workload I decided against it.

## Task 2 design decisions

### Storing of the wishlist

Wishlist is stored in a repeated integer property (as it is an id) on the Profile.

### Adding a session to the wishlist

Ideally the addSessionToWishlist would take a websafe key of the session, but to make testing easier, it takes an id of the session and a websafe conference key of the parent. Both of these parameters are required as they're both needed to correctly pull session from the datastore without the key. 

## Task 3 design decisions

### Two extra queries

I have added two additional queries that can be accessed through getSessionsByLocation(location) and getSessionsByDate(date) which filter all sessions by location and date accordingly. These would allow user to filter down all the sessions to see the one they may be interested in.

### Query problem

The problem with having query for both non-workshop and before 7pm sessions is that Google Datastore doesn't allow multiple inequalities for different properties. Possible solution would be to make two seperate queries and combine them in the code. It would in fact be better to have a single query (hitting the datastore only once) and filtering down within Python. I would probably do non-workshop filtering in the query and time filtering in Python. 

## Task 4 design decisions

### Featured speaker

Featured speaker is stored in memcache. When new session is being added, we iterate through all sessions and find the speaker who has the most sessions. If there is a tie, only one user will be shown. Iterating over all sessions is fairly resource heavy so it was implemented as a task to allow for no extra time delay when serving a createSession endpoint. Since featured speaker will be stored in memcache, there will be no datastore queries made at the time of calling getFeaturedSpeaker.
