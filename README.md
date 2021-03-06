App Engine application for the Udacity training course.

## Products
- [App Engine][1]

## Language
- [Python][2]

## APIs
- [Google Cloud Endpoints][3]

## Setup Instructions
1. Update the value of `application` in `app.yaml` to the app ID you
   have registered in the App Engine admin console and would like to use to host
   your instance of this sample.
1. Update the values at the top of `settings.py` to
   reflect the respective client IDs you have registered in the
   [Developer Console][4].
1. Update the value of CLIENT_ID in `static/js/app.js` to the Web client ID
1. (Optional) Mark the configuration files as unchanged as follows:
   `$ git update-index --assume-unchanged app.yaml settings.py static/js/app.js`
1. Run the app with the devserver using `dev_appserver.py DIR`, and ensure it's running by visiting your local server's address (by default [localhost:8080][5].)
1. (Optional) Generate your client library(ies) with [the endpoints tool][6].
1. Deploy your application.


[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
[4]: https://console.developers.google.com/
[5]: https://localhost:8080/
[6]: https://developers.google.com/appengine/docs/python/endpoints/endpoints_tool

# APP:
Name: mozgul-udacity-fs-p4  
Access: [mozgul-udacity-fs-p4.appspot.com](https://mozgul-udacity-fs-p4.appspot.com)

## Task 1:

#### Design Decisions:  
#####Profile 1--* Conferences
Profile has many (created/owned) conferences. Conference belongs to one profile (creator)

#####Conference 1--* Sessions
Conference has many sessions. Session belongs to one conference

#####Speaker:
Option 1) Implemented this option. Speaker is a (string) property of session.  
If app business logic is not focusing on speakers (we are not keeping other information about the speaker other than name), faster implementation, faster read time for sessions and grabbing the speaker name. But duplicating speaker data (name).    
  
Option 2) NOT Implemented this option. Speaker as a seperate entity. Use key (or keys) to associate with session. Can store more information about speakers. A single update to a speaker is reflected to every related session, for example no need to iterate through every session object to fix a name typo for speaker.

#####Wishlist:
Wishlist is a repeated string property field, containing keys of sessions.


```py
class Profile(ndb.Model):
    """Profile -- User profile object"""
    displayName = ndb.StringProperty()
    mainEmail = ndb.StringProperty()
    teeShirtSize = ndb.StringProperty(default='NOT_SPECIFIED')
    conferenceKeysToAttend = ndb.StringProperty(repeated=True)
    sessionKeysWishlisted = ndb.StringProperty(repeated=True)

class Session(ndb.Model):
    """Session -- Session object"""
    name = ndb.StringProperty(required=True)
    speaker = ndb.StringProperty()
    sessionType = ndb.StringProperty()
    startTime = ndb.StringProperty()
    duration = ndb.IntegerProperty()
    highlights = ndb.StringProperty()

class SessionForm(messages.Message):
    """SessionForm -- Session outbound form message"""
    name = messages.StringField(1)
    speaker = messages.StringField(2)
    sessionType = messages.StringField(3)
    startTime = messages.StringField(4)
    duration = messages.IntegerField(5)
    urlsafe_id = messages.StringField(6)
    highlights = messages.StringField(7)
```  


## Task 3: 

### Query Problem

#### Question:  

Let’s say that you don't like workshops and you don't like sessions after 7 pm. How would you handle a query for all non-workshop sessions before 7 pm? What is the problem for implementing this query? What ways to solve it did you think of?  

#### Answer:  
From [docs](https://cloud.google.com/datastore/docs/concepts/queries#Datastore_Restrictions_on_queries):  
``` 
Inequality filters are limited to at most one property.  

To avoid having to scan the entire index, the query mechanism relies on all 
of a query's potential results being adjacent to one another in the index. 

To satisfy this constraint, a single query may not use inequality comparisons 
(LESS_THAN, LESS_THAN_OR_EQUAL, GREATER_THAN, GREATER_THAN_OR_EQUAL) on more 
than one property across all of its filters.
```

#### Workarounds:  
Option 1) Apply one inequality filter, do the rest of the filtering by manually iterating  
Option 2) Enumerate possible types (lecture, workshop, training, qa) and use equalities or IN operator for the types other than the undesired ones


### Additional Queries:
1. **getSessionsWithMissingData**: Query for all the sessions that has missing properties. Used for quality user experience, these incomplete sessions can be fixed or deleted.  
2. **getLightningTalks**: Query for all the sessions that are between 5-20 mins long.
