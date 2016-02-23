Conference Central
==================

## Summary

Conference Central registration web app allows users to create new conferences, sessions, speakers and provides functionality for users to add a session to their wishlist or to attend.

All the functionality is implemented in a web service architecture so that it is independent for a particular platform (e.g web) and can be easily extended to different platforms (e.g Android, iOS). The app's full functinoality can be tested through APIs explorer.

This project is done as a part of Udacity Full Stack Web Development Nanodegree program.

## Products

- [App Engine][1]

## Language

- [Python][2]

## APIs

- [Google Cloud Endpoints][3]

## Access

#### App ID
ud858-subhash-playground

#### Front-end
https://ud858-subhash-playground.appspot.com

From here you can navitage to different pages and perform the provided actions.

#### APIs explorer
https://ud858-subhash-playground.appspot.com/_ah/api/explorer

From here you can select the 'Conference API v0.1' which will bring a list of all available APIs. This list allows you to play with different apis that front-end applications use. For each api, provide input as required and click the button 'Authorize and execute', it should ask for authentication and then return the appropriate results

## Project tasks design decisions

### Task 1 : Add Sessions to a Conference

As a part of this task I added two new entities and different apis to the project

#### Entities
**Session** : Created Sessions entity with an ancestor relationship to Conference as Sessions cannot exist on their own and are totally dependent on the Conference. The ancestor relationship will guarantee the strong consistency between Conference and Sessions.

```
class Session(ndb.Model):
    """Session -- Session object"""
    name = ndb.StringProperty(required=True)
    highlights = ndb.StringProperty(repeated=True)
    speaker = ndb.KeyProperty(required=True)
    duration = ndb.IntegerProperty()
    typeOfSession = ndb.StringProperty(default='NOT_SPECIFIED')
    date = ndb.DateProperty()
    startTime = ndb.TimeProperty()
``` 

**Speaker** : Speaker information is stored in its own entity because it gives flexibility in future to add any new information about a speaker or to add different functionality that might help our users. Also storing only the name could lead to duplicate name collisions.

```
class Speaker(ndb.Model):
    """Speaker --- speaker object"""
    name = ndb.StringProperty(required=True)
    company = ndb.StringProperty()
    email = ndb.StringProperty()
    phone = ndb.StringProperty()
    websiteUrl = ndb.StringProperty()
    sessionKeys = ndb.KeyProperty(repeated=True)
```

#### APIs

* createSession - Create and return Session object
* getConferenceSessions - Get list of Conference sessions
* getConferenceSessionsByType - Get list of Conference sessions of a particular type
* getSessionsBySpeaker - Get list of Session associated with a Speaker
* createSpeaker - Create and return Speaker object
* getSpeakers - Get list of Speakers

### Task 2 : Add Sessions to User Wishlist

For this task modified the Profile entity to also keep track of User's Sessions wishlist and added APIs that allow user to add/remove from their wishlist

#### Entity

```
class Profile(ndb.Model):
    """Profile -- User profile object"""
    displayName = ndb.StringProperty()
    mainEmail = ndb.StringProperty()
    teeShirtSize = ndb.StringProperty(default='NOT_SPECIFIED')
    conferenceKeysToAttend = ndb.StringProperty(repeated=True)
    sessionsWishlist = ndb.StringProperty(repeated=True)
```

#### APIs

* addSessionToWishlist - Add Session to user's wishlist
* removeSessionFromWishlist - Remove Session from user's wishlist
* getSessionsInWishlist - Get list of Sessions in user's wishlist

### Task 3 : Work on indexes and queries

As a part of this task added 4 additional queries that help user to get the list of Sessions with a particular requirement

* getSessionsOfTypeAndStarttime - Get list of Sessions of given type and start time
* getSessionsOnDateSortByDuration - Get list of Sessions in the given date sorted by duration
* getSessionsOnDateSortByStartTime - Get list of Sessions in given date sorted by start time
* getSessionsOfNotTypeAndStarttime - Get list of Session of not type and before start time. Sample for double inequality

Double inequality query:
Datastore always uses the indexes to find the matching data. Inorder to match the performace requirements it adds a query restriction where an inequality filter can be applied to atmost one property. For the sample query we need to apply inequality on two properties SessionType and startTime. In order to overcome the restriction modfied the SessionType property from inequality filter to an equality filter. As SessionType is a ENUM set and as it is finite, I turned the inequality to a equality on other ENUM values with ndb.OR.

NOTE: Most of the Session are added to the default date 2015-06-06, so please use this date for the above query apis. Or you can create Sessions on your own and verify the above queries.

Also as a part of this task updated index.yaml inorder to support the above queries. 
```
- kind: Session
  properties:
  - name: date
  - name: duration

- kind: Session
  properties:
  - name: date
  - name: startTime

- kind: Session
  properties:
  - name: typeOfSession
  - name: duration

- kind: Session
  properties:
  - name: typeOfSession
  - name: startTime
```

### Task 4 : Add a task

When a Session is created, added a task to the default push queue to check if the speaker has multiple sessions in a conference if so update the memcache to feature that speaker. The current featured speaker can be seen through the below api

* getFeaturedSpeaker - Get the featured speaker from memcache

[1]: https://developers.google.com/appengine
[2]: http://python.org
[3]: https://developers.google.com/appengine/docs/python/endpoints/
