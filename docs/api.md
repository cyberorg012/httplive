In order to reduce the size, httplive provides you two min.js, one is using webworker, the other is not.If you are using webpack as a building tool, you can simply switch between them.

```
resolve:{
    alias:{
        httplive:path.resolve(__dirname,'node_modules/httplive/dist/index.min.js'),
        workerlive:path.resolve(__dirname,'node_modules/httplive/dist/index.worker.min.js'),
    }
}
```

Otherwise, you can import them by path:

```
// no-worker
import HTTPLive from 'httplive';

// worker
import HTTPLive from 'httplive/dist/index.worker.min';
```

And, no matter which one you import, their interface is the same.

## Import

```
import HTTPLive from 'httplive';
```

### Construtor

Its constructor is:

```
[Constructor]
interface HTTPLive(params)

[Parameter]
Object params{
    attribute HTMLElement video;[necessary]
    attribute String      url;
    attribute Object      mse;
    attribute Object      request;
}

[SubParameter]
Object mse{
    attribute Number    maxBufferTime, 60;
    attribute Number    trailedTime, 2;
    attribute Boolean   keepUpdated, true;
    attribute Number    playbackRate, 1.5;
}
```

For above params, only the video is necessary. 

 - video: The video element you want to bind with [MSE](https://www.w3.org/TR/media-source/#).
 - url: the link of live video
 - mse: some params to control the MSE
    - maxBufferTime: the maximum playing time to remove timeRanges
    - keepUpdated: indicate to catch up time when the video time behind the ranges.end(0)
    - trailedTime: the maximum delay time of video behind the ranges.end(0), otherwise, the play will use fast forward to catch up live time. 
    - playbackRate: the fast forward rate when to catch up time.

For better experience, you can canel the `keepUpdated` operation.

```
let video = document.getElementById('video');
let flv = new HTTPLive({
        video
    });

let flv = new HTTPLive({
    video,
    mse:{
        keepUpdated: false
    }
});
```


### Initialization

```
[Return]
interface HTTPLive(params){
    void send(String url);
    void retry();
    EventListener on(event,fn);
    EventListener bind(event,fn);
    EventListener addEventListener(event,fn);
    
    static boolean isSupported();
}

enum event{
    info,
    sync
}
```

 - send(url): when you start to recevice the live video chunk, you can call this fn. but remember that, you can only trigger this fn once.
 - retry(): provide you a way to retry the connection.
 - on/bind/addEventListener(): just add some listener to get some valuable messages.
 - isSupported: check if the browser support MSE.

### Event

Httplive provide two events for you, `sync` and `info`. You can get the Initialization Info and  palyback info about A/V sync.

```
flv.on('info',msg=>{
    window.__report({
        mediaInfo
        videoMime
        audioMime
    });
  
    window.__report("The video size, width: " + mediaInfo.width + " height: " + mediaInfo.height);
})

flv.on('sync',msg=>{
    let {diffTimebase,audioTimebase,videoTimeStamp} = msg;
    
    if(diffTimebase > 100){
        // videoTime is 100ms faster than audio
    }else{
        // A/V is sync
    }
})
```

The `info` detail is :

```
Event info{
    attribute Object mediaInfo；
    attribute String videoMime;
    attribute String audioMime;
}

Object mediaInfo{
    attribute Number audiocodecid:
    attribute Number audiodatarate
    attribute Number audiosamplerate
    attribute Number audiosamplesize
    attribute Number duration
    attribute String encoder
    attribute Number filesize
    attribute Number height
    attribute Boolean stereo
    attribute Number videocodecid
    attribute Number videodatarate
    attribute Number width
    attribute Boolean hasVideo
    attribute Boolean hasAudio
}
```

the `sync` deatil is:

```
Event sync{
        attribute Number videoTimebase;
        attribute Number audioTimebase;
        attribute Number diffTimebase;
        attribute Number videoTimeStamp;
        attribute Number audioTimeStamp;
}
```