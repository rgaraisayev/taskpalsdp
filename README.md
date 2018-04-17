# TaskPal application
### Short Introduction
  Mobile application that connects people who do the same things by gamifying their tasks. It is designed with the intention to help people refocus on what really matters. Think of it as social network but for those who struggle to get things done.

# Table of content

* What is it?
* Implementation
  * Technical Implementation overview 
  * Android Implementation overview
  * Backend Design&Implementation
    * Rest API
    * Database
    * Text Processing
* Conclusion


# What is it?

  Taskpal is mobile application, currently supports only android OS. Main feature is timer. It requires signing in with either Google or Facebook. After signing in, users start doing their tasks by the help of implemented timer. In main screen simple tasks are located to warm up. After spesific level custom task adding is activated. Users are able to see their and their taskpals'(friends) activities, and leaderboard through the app. 

Happy Tasking !


# Implementation

## Technical Implementation overview 
  Current version of TaskPal is built on Android platform. NodeJs is used for backend programming, for storing data, MongoDB is used as NoSQL database, Amazon Elastic Computing for cloud (AWS EC2), so the database and web server are located there.Facebook SDK, Google SDK are used for login. In later sections more information is provided.

## Client implementation (Android)
  The main functionality of the application is Timer, implemented with the use of android background service. Service is in 'START_STICKY' mode, so it is supposed to work even the app is closed. If user leaves app while doing task, notifcation will pop up showing visual progress of task time, and the time user is spending outside of app. Below some screenshots added.
  
  
  
  
  In current version of TaskPal, users are able to create or join existing tasks, make new taskpals, visit profiles of those taskpals, collect points doing tasks, and find new taskpals that have similar interests with them. Users can adjust the time of their task. Currenly time can be chosen from 1, 5, 10, 15, 20 minutes.
