# Vuejs-Django-Chanel
Configuration vue js et django chanel

## Install Django channels

`pip install -U channels`

## routing projet

```bash
from channels.auth import AuthMiddlewareStack
from channels.routing import ProtocolTypeRouter, URLRouter
import chat.routing

application = ProtocolTypeRouter({
    # (http->django views is added by default)
    'websocket': AuthMiddlewareStack(
        URLRouter(
            chat.routing.websocket_urlpatterns
        )
    ),
})


```

## Routing chat app

```bash

# chat/routing.py
from django.urls import path

from . import consumers

websocket_urlpatterns = [
    path('ws/chat/hello/', consumers.ChatConsumerEditor),
    path('ws/chat/salon/', consumers.ChatConsumerGroup),
    path('ws/suggest/suggestion/', consumers.ChatSuggestion),

```

## Intall redis django

`pip3 install channels_redis`

## Config Redis

```bash

# Channels
ASGI_APPLICATION = 'mysite.routing.application'
CHANNEL_LAYERS = {
    'default': {
        'BACKEND': 'channels_redis.core.RedisChannelLayer',
        'CONFIG': {
            "hosts": [('127.0.0.1', 6379)],
        },
    },
}

```



## Chat consumer

```bash

# chat/consumers.py

from channels.generic.websocket import AsyncWebsocketConsumer
import json

from .models import *

class ChatConsumerEditor(AsyncWebsocketConsumer):
    async def connect(self):
       
       # self.room_name = self.scope['url_route']['kwargs']['room_name']
        self.room_name = "hello"
        self.room_group_name = 'chat_%s' % self.room_name

        # Join room group
        await self.channel_layer.group_add(
            self.room_group_name,
            self.channel_name
        )

        await self.accept()

    async def disconnect(self, close_code):
        # Leave room group
        await self.channel_layer.group_discard(
            self.room_group_name,
            self.channel_name
        )
        

    # Receive message from WebSocket
    async def receive(self, text_data):
        text_data_json = json.loads(text_data)
        message = text_data_json['message'] 
        code = text_data_json['code'] 
        admin = text_data_json['admin'] 
        print(code)

        # Send message to room group
        await self.channel_layer.group_send(
            self.room_group_name,
            {
                'type': 'chat_message',
                'message': message,
                'code':code,
                'admin':admin
            }
        )

    # Receive message from room group
    async def chat_message(self, event):
        message = event['message']
        code = event['code']
        admin = event['admin']
        
        
        # messages = TestMessage(message = message)
        # messages.save()

        # Send message to WebSocket
        await self.send(text_data=json.dumps({
            
            'code': code,
            'message': message ,
            'admin':admin
            
        }))


```


## Vue js ws

```bash
<script src="https://cdnjs.cloudflare.com/ajax/libs/vue/2.5.17/vue.min.js"></script>
     <script src="https://cdn.jsdelivr.net/npm/reconnecting-websocket@4.1.10/dist/reconnecting-websocket-cjs.min.js"></script>
    
      <script src="https://cdn.jsdelivr.net/npm/vue-websocket@0.2.3/dist/vue-websocket.min.js"></script>
      
```

## Code js


```bash

const app = new Vue({
        el: "#app",
        data: {
            message: "",
        

        },
        delimiters: ["${", "}"],
        mounted: function() {
            
            this.connect()
            
        },
        methods: {
           
            connect() {
                roomName = "hello"
                user = "{{ user.username }}"
                this.socket = new WebSocket('ws://' + window.location.host +'/ws/chat/' + roomName + '/');
                this.socket.onopen = () => {
                    this.status = "connected";
                    console.log("connecte")
                    
                    this.socket.onmessage = ({data}) => {
                        
                     # Recuperation message      
                   
                    };
                };
            },
            disconnect() {
                this.socket.close();
                
                console.log("deconnecte")
            },
            
            
            teatsend: function(){
                
                    this.socket.send(JSON.stringify({ 'message':"code send succes", 'admin':this.admin, 'code':this.codedep }));
                    
                    
                })
                
                    
            },
            updateAdmin: function(){
                
                this.socket.send(JSON.stringify({ 'message':"code send succes", 'admin':this.admin, 'code':this.codedep }));

            },
            
                     
        }
});



```
