# Galactus Phase 4

## Overview and Goals

Galactus is an online video syncing service that allows multiple users to watch
and control the same videos from the most common video streaming services at the
same time. Our goal is to provide a blazing fast syncing service that can allow
more than 128 users to watch the same video at the same time.

[Demo Video - YouTube](https://www.youtube.com/watch?v=89ioAbGyJAw)

[Discussions around Project Goals, Scalablity, and What's Next for Galactus! - YouTube](https://www.youtube.com/watch?v=8YLeX4N_X98)


## Team Members

- Luke Yeh
- Dan Cline
- Sathvik Birudavolu
- Alex Hulbert
- Ayush Khandelwal

## System Design

Our system is composed of 4 parts.

1. **Frontend**. The frontend is the front facing service for our clients. This
   is where users can watch and control a shared video session.
2. **Room Service**. This service manages our virtual room/sessions. It's job as
   a microservice is to create rooms, add users to rooms, and delete rooms.
4. **Queue Service**. This service managers the video queue of a specific room.
   Users in a room can queue videos that they want to watch in the future.
5. **Sync Service**. The sync service makes sure that users in the same room are
   seeing the same video at the right speed and timestamp.

### System Design Diagram

![](https://i.imgur.com/Lql8uEu.png[/img])

### Frontend

The front-end uses [NextJS](https://nextjs.org/). The frontend has two pages.

1. **Index Page**. The index page allows users to create a new room or join an
   existing room.
2. **Room Page**. The room page is where the video player exists. This is where
   users can enter the URL to a video that they want to watch. The video player
   is completely independent of whatever video service the video is being played
   on. Users have full control of the timestamp and speed of the video through
   our player.

The video player is able to retrieve videos from the following video streaming
services:

    - YouTube
    - SoundCloud
    - Facebook
    - Vimeo
    - Twitch
    - Streamable
    - Wistia
    - DailyMotion
    - Mixcloud
    - Vidyard
    - MP4 Files

All the user needs to do is simply enter the URL of the video into the video
player, and our system deals with everything.

Within our player, we have 3 callbacks that connect to the syncing service. When
these closures are triggered, the syncing service will make sure that the same
actions are applied to everyone in the room. Read the syncing service section
for more.

1. **onSeek**. Called whenever any user changes the position of the video
   slider.
2. **onPlay**. Called whenever any user clicks on the play button.
3. **onPause**. Called whenever any user clicks on the pause button

### Room Service

The room service has a single endpoint called `addRoom`. This endpoint simply
creates a room. We store a room in memory as the following structure:

```go
// Room A room
type Room struct {
	Id        int64     `json:"id,omitempty"`
	Code      string    `json:"code,omitempty"`
	CreatedAt time.Time `json:"createdAt,omitempty"`
}
```

Other service use the `Code` as a reference point for syncing and queueing.

For more information please consult the repo readme for this [service](https://www.github.com/galactus-player/roomservice)


### Queue Service

The service handles the task of managing the queue of videos for rooms. It let's users add, remove and modify the ordering of videos in queue, given a room code such as `1234`. The service also does the job of adding metadata such as thumbnail url and title when adding a url of a video to the queue.

The [queueservice](https://www.github.com/galactus-player/queueservice) has 4 enpoints: 
```
(GET)  /v1/queue/{room_code}          # Retrieves list of video in the right order
(POST) /v1/queue/{room_code}/add     # Adds video
(POST) /v1/queue/{room_code}/remove  # Removes video
(GET)  /v1/queue/{room_code}/play     # Moves video to top of queue
```

#### Video Object

```typescript
export interface Video {
    id: string;
    index: number;
    url: string;
    thumbnailUrl?: string;
    title?: string;
    addedAt: Date;
}
```

For more information please consult the repo readme for this [service](https://www.github.com/galactus-player/queueservice)


### Sync Service

The sync service allows for users in the same room to have the same video shown,
and have it at the same timestamp.

```typescript
export default function initConnection(socket: Socket) {
  socket.on("sync", (req) => sync(socket, req));
  socket.on("status", () => status(socket, false));
  socket.on("pause", (action) => setPlaying(socket, action, false));
  socket.on("play", (action) => setPlaying(socket, action, true));
  socket.on("seek", (action) => seek(socket, action));
  socket.on("reset", reset);
}
```
For more information please consult the repo readme for this [service](https://www.github.com/galactus-player/sync-service)


## How is this Scalable
#### Micro service centric design
The nature of the system design of our product allows for scalability. Each
service can operate on its own independently from other services. So if one
service goes down, the other services will still remaining running.


Each service can be called by a HTTP request. If a certain HTTP request fails,
our services have elaborate error handling code that allows other services to
not completely crash and burn.

#### Container and Container Orchestration

#### Health checks
Our microservices may need to be restarted in the case of failure, so we implement health checks which monitor the status of the running containers. This allows the containers to be restarted if their status ever changes from being in a healthy state to being in an unhealthy state.
Additionally, because these microservices are dependent on systems such as databases, healthchecks allow us to initialize our system in the correct order every time.

For example, the databases (postgres and mongodb) are started before every other service and marked as "healthy" when a database client is able to connect with the service running. 

## Individual Videos
1. [Luke](https://youtu.be/HtpxbTku9cM)
2. [Dan](https://www.twitch.tv/videos/811546509)
3. [Sathvik](https://www.youtube.com/watch?v=O9mKNa0-wz4)
4. [Ayush](https://example.com)
5. [Alex](https://youtu.be/YY7kdNI0adk)

## Video Discussion


## Code

1. [Web Player](https://www.github.com/galactus-player/web)
2. [Room Service](https://www.github.com/galactus-player/roomservice)
3. [Queue Service](https://www.github.com/galactus-player/queueservice)
4. [Sync Service](https://www.github.com/galactus-player/sync-service)
5. [Front-End](https://github.com/Galactus-Player/frontend)
