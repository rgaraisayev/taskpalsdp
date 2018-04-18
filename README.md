# TaskPal application
### Short Introduction
  Mobile application that connects people who do the same things by gamifying their tasks. It is designed with the intention to help people refocus on what really matters. Think of it as social network but for those who struggle to get things done.

# Table of content

* [What is it?](#what-is-it)
* [Implementation](#implementation)
  * [Technical Implementation overview](#technical-implementation-overview)
  * [Android Implementation overview](#android-implementation-overview)
  * [Backend Implementation](#backend-implementation)
    * [Rest API](#rest-api)
    * [Database](#database)
    * [Text Processing](#text-processing)
* [Conclusion](#conclusion)


# What is it?

  Taskpal is mobile application, currently supports only android OS. Main feature is timer. It requires signing in with either Google or Facebook. After signing in, users start doing their tasks by the help of implemented timer. In main screen simple tasks are located to warm up. After spesific level custom task adding is activated. Users are able to see their and their taskpals'(friends) activities, and leaderboard through the app. 

Happy Tasking !


# Implementation

## Technical Implementation overview 
  Current version of TaskPal is built on Android platform. NodeJs is used for backend programming, for storing data, MongoDB is used as NoSQL database, Amazon Elastic Computing for cloud (AWS EC2), so the database and web server are located there. Facebook SDK, Google SDK are used for login. In later sections more information is provided.

## Client implementation (Android)
  The main functionality of the application is Timer, implemented with the use of android background service. Service is in 'START_STICKY' mode, so it is supposed to work even the app is closed. If user leaves app while doing task, notifcation will pop up showing visual progress of task time, and the time user is spending outside of app.
  
When service gets started first time by app or by android system after it app is killed(cleared from recents), `timer()` function is called. 
  ``` @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        timer();
        return START_STICKY;
    }    
  ```
`timer()` function is where whole timer logic is implemented. Inside `timer()` function, `java.util.Timer` class is used to schedule execution at fixed rate. Every 1 second it updates UI(if app is visible to user), updates time, notification(if showing), in every 2-2.5 seconds updates local database data(task), and every 5-7 seconds it syncs local data with server using rest API. Server sync is coded to save last possible state of task(spent seconds etc) in case of lost internet connection or other possible issues.

```
 timer.scheduleAtFixedRate(new TimerTask() {
            long localUpdateTime;
            long remoteUpdateTime;

            @Override
            public void run() {
             LEFT_TIME = LEFT_TIME <= 0 ? 0 : LEFT_TIME - 1; // decrease time(secods) by one
             
             /* other code implementation goes below */
             
             /* when time finishes, change state, and show notification or if it goes on update local db in every 2.5 secs */
             if (LEFT_TIME == 0) {
                    updateTaskTry(state_finish);
                    state = state_finish;
                    showNotification();
                } else {
                    if (System.currentTimeMillis() - localUpdateTime > 2_500) {
                        localUpdateTime = System.currentTimeMillis();
                        updateTaskTry(state_running);
                    }
                }
                
                
             /* other code implementation goes below */
             
            }
        }, 0, 1000);// every tick is 1 second

```

Inside activity that is visible to user, we have to show updated progress and time. In order to do this, `android.content.BroadcastReceiver` extended and registered to that particular activity.

```
  private class TimerUpdateReceiver extends BroadcastReceiver {
        @Override
        public void onReceive(Context context, Intent intent) {
         int TOTAL_TIME = intent.getIntExtra("TOTAL_TIME", 0);
         int LEFT_TIME = intent.getIntExtra("LEFT_TIME", TOTAL_TIME);
         
         binding.time.setText(String.format(timeFormat, LEFT_TIME / 60, LEFT_TIME % 60));
         float animPos = (100 * (TOTAL_TIME - LEFT_TIME)) / TOTAL_TIME;
         binding.taskTimer.setProgress((int) animPos);
         
         /* other code implementation goes below */
       }
    }
```

This receiver is registered `onResume()` life circle method, and unregistered in `onPause()` life circle method. Meaning `onReceive()` method will be executed continuously starting from `onResume()` call. In order to prevent memory leak, we unrelease(unregister) it when activity goes on pause mode, in other words `onPause()` is called.

```
    @Override
    public void onResume() {
        super.onResume();     
        TaskActivity.this.registerReceiver(receiver, intentFilter);
         /* other code implementation goes below */
    }
    
    @Override
    public void onPause() {
        super.onPause();       
        TaskActivity.this.unregisterReceiver(receiver);
          /* other code implementation goes below */
    }

```
<hr/>

Below you can see simplified user flow diagram:
<div align="center">
<img src="https://github.com/rgaraisayev/taskpalsdp/blob/master/screens/User flow diagram of Taskpal.jpg"  width="500" height="450" />
</div>

<hr/>

Below some screenshots are provided. Task page, Reward Page, Leaderboard and activities, Home page.

<div align="center" display="inline-block">
  <img src="https://github.com/rgaraisayev/taskpalsdp/blob/master/screens/photo5379957081458714850.jpg"  width="200" height="350" />
  <img src="https://github.com/rgaraisayev/taskpalsdp/blob/master/screens/photo5379957081458714849.jpg"  width="200" height="350" />
  <img src="https://github.com/rgaraisayev/taskpalsdp/blob/master/screens/photo5379957081458714848.jpg"  width="200" height="350" />
  <img src="https://github.com/rgaraisayev/taskpalsdp/blob/master/screens/photo5379957081458714847.jpg"  width="200" height="350" />  
</div>
  
  <br/>
  <br/>
  
  In current version of TaskPal, users are able to create or join existing tasks, make new taskpals, visit profiles of those taskpals, collect points doing tasks, and find new taskpals that have similar interests with them. Users can adjust the time of their task. Currenly time can be chosen from 1, 5, 10, 15, 20 minutes. After finishing every task, they are rewarded with coins, trophies, points and one new taskpal - other user who is doing similar tasks and having similar interest with him/her.
  
  
  
## Backend Implementation
   [NodeJs](https://nodejs.org), combined with [MongoDB](https://www.mongodb.com/) is used for programming restful API and database. As it is known, NodeJs is very flexible in terms of  development and deployment. And we favored NoSql over Relational Databases for some advantages such as easy to change schema(schema-less), no need for predefined structure of data etc,  Because of the schema agnostic nature of NoSQL databases, they’re very capable of managing change. In case of change, such as adding new or removing field in schemas, we don’t have to rewrite ETL routines, application code changes is enough to apply new changes to database collections. 
   
### Rest API
  Rest API is build using Nodejs runtime which is build on Chrome's V8 JavaScript engine. Programming language is JavaScript, but [expressjs](https://expressjs.com/) is used as framework to NodeJs which helps to handle MVC(view templates like [EJS](http://embeddedjs.com/), [Jade](http://jade-lang.com/), etc), routes, etc.
<br/>
Below code snipped shows registering routes and starting/listening to pedefined port.
```
app.use('/taskpal/api/v1.0/users', require('./api/versions/v1.0/routes/users'));
app.use('/taskpal/api/v1.0/tasks', require('./api/versions/v1.0/routes/tasks'));
app.use('/taskpal/api/v1.1/users', require('./api/versions/v1.1/routes/users'));
app.use('/taskpal/api/v1.1/tasks', require('./api/versions/v1.1/routes/tasks'));
app.use('/taskpal/api/report', require('./api/general/routes/report'));

const port = process.env.PORT | 8081
app.listen(port)
```
<br/>
To develop this rest api, below external libraries are used, use of each of them is explained in front of them.
<br/>

```
 "dependencies": {
    "body-parser": "^1.18.2", // This is required to parse incoming request bodies in a middleware
    "date-diff": "^0.1.3", // This library used to compare two date and get difference in seconds, minutes, hours and so on
    "ejs": "^2.5.8", // This is view template, it is used to configur and develop little dashboard to add/edit/remove tasks/levels
    "express": "^4.16.2", // Express framework
    "express-promise-router": "^2.0.0", // Express related, required for router implementation
    "fs-extra": "^4.0.2", // This library used for reading and writing files to filesystem both asyc/sync
    "mongoose": "^4.13.5", // Mongoose is mongodb object modeling for node.js
    "morgan": "^1.9.0", // Logging
    "node-schedule": "^1.2.5", // executing periodic jobs
    "randomstring": "^1.1.5" // generating random string, used for generating task promo code(not used in this version)
  }
  
```
<br/>

## Database

MongoDb is used to handle database. Mongoose mongodb object modelling used for making database operations simple. This is example mongoose schema, it represents `level` collection in database.

```
const mongoose = require('mongoose');
const Schema = mongoose.Schema;

const level = new Schema({
    level: Number,
    description: String,
    pointsToCollect: Number,
    bonusCoins: Number,
    bonusPoints: Number,
    dateWritten: {
        type: Date,
        default: Date.now()
    },
    dateUpdated: Date,
    status: {
        type: String,
        default: 'active',
        enum: ['active', 'deactive']
    }

});

module.exports = mongoose.model('level', level);
```
<br/>
Mongoose allows us to save/edit/delete/find data from mongo database very easily. For example retrieving user looks like this 
`var myLevel = await level.findById(userLevelId)` - it will return user level by its id.
<br/>

## Text Processing - Task similarity
  One of the important features of Taskpal is matching and connecting users with same interests. To do this, we get the help of document similarity API from Marketplace (https://market.mashape.com/). We tried various text similarity APIs and tested them with short and long texts. Our success criteria for text similarity is that it should work on mainly short texts, because we have to compare task names which contain 3 to 4 words. Currently, we are searching for libraries that are customizable, meaning, models for parsing text and finding semantic similarity can be adjusted to our needs. Levels of similarity should be definable, such as similar, direct and indirect similarity. For example, “Learning Java programming language basics” and “learning Swift programming language basics” are similar in terms of learning, but objective is completely different, meaning Swift and Java developer’s interests may not be same. So this could fall into indirect similarity category. However, current API considers them as complete similar. Since this is problem, we are planning to build our own text processing model.
  
<br/>
This is simple visual representation of how algorithm should work to mark similar people:
<br/>
<img src="https://github.com/rgaraisayev/taskpalsdp/blob/master/screens/tpSteps.jpg"/>
    

   
# Conclusion

In conclusion, we developed beta version of TaskPal application. Almost all features are implemented and we are continuosly improving design, quality and functionality. Still much work is needed to be done.
   
  
  
  
