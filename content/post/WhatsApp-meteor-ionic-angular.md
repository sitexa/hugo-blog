+++
title = "Build WhatsApp with Meteor , Ionic 2 and Angular 2"
description = "Build WhatsApp with Meteor , Ionic 2 and Angular 2."
tags = [
    "Meteor",
    "Ionic",
    "Angular",
    "MongoDB",
]
date = "2017-11-28"
categories = [
    "Development",
    "nodejs",
]
thumbnail = "images/meteor-guide/whatsapp2.png"
+++

开发一个应用程序项目，有web端，手机端，或者两者都要。要求采用最好的平台和框架，能快速启动，并且要求在项目增长的过程中，相应的平台和
和技术都能快速发展，且有丰富的社区支持。

<!--more-->

**Note**

-   WhatsApp Clone with Meteor CLI and Ionic - https://github.com/Urigo/Ionic-MeteorCLI-WhatsApp
-   WhatsApp Clone with Ionic 2 and Meteor CLI (Last Update: 14.02.2017) - https://github.com/Urigo/Ionic2-MeteorCLI-WhatsApp
-   WhatsApp Clone with Meteor and Ionic 2 CLI (Last Update: 2017-06-15) - https://github.com/Urigo/Ionic2CLI-Meteor-WhatsApp

目前最好的实践是采用Meteor+Ionic2+Angular2。

**Angular**是一个前端平台，它包括你建立Angular应用所需要的前端部分的所有技术。Angular也有自己的客户端cli，它是基于webpack的。

**Ionic**是基于Angular的平台。在快速开发跨平台的混合(hybrid)移动应用中，它已经成为一个最流行的解决方案。

Ionic平台包括用于原型、构建、测试、部署应用程序的解决方案、启动应用程序的市场、插件和主题、CLI集成和推送通知服务。

**Meteor**，但是，你的应用程序需要一个完整的全栈解决方案。

Meteor已成为唯一的开源Javascript平台, 为您创建一个实时的移动连接的应用程序提供全套的解决方案。

Meteor是可靠的、快速的、易于开发和部署的平台。当你的应用程序随着时间的推移而变得复杂且规模增长时，Meteor将帮助你处理各种复杂的局面。


## WhatsApp : Ionic 2 + Meteor Cli


## WhatsApp : Ionic 2 Cli + Meteor

### (一)Setup the application

####    (1) npm install -g ionic cordova

node version >= 5, here my version is : npm = 5.5.1, node = 9.2.0, cordova = 7.1.0

####    (2) ionic start whatsapp blank --cordova --no-link

add src/declarations.d.ts

``` 
/*
  Declaration files are how the Typescript compiler knows about the type information(or shape) of an object.
  They're what make intellisense work and make Typescript know all about your code.
  A wildcard module is declared below to allow third party libraries to be used in an app even if they don't
  provide their own type declarations.
  To learn more about using third party libraries in an Ionic app, check out the docs here:
  http://ionicframework.com/docs/v2/resources/third-party-libs/
  For more info on type definition files, check out the Typescript docs here:
  https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html
*/
```

####    (3) ionic serve

![](/images/meteor-guide/ionic-setup.png)

####    (4) npm install --save typescript@~2.4

add webpack config declaration in package.json

``` 
"config": {
    "ionic_webpack": "./webpack.config.js"
  }
```

```cp node_modules/@ionic/app-scripts/config/webpack.config.js .```

add ability to the config file:

-   load external ```TypeScript``` modules
-   an alias for our ```Meteor``` server under the ```api``` dir 
-   import ```Meteor``` packages and ```Cordova``` plugins

``` 
alias: {
      'api': path.resolve(__dirname, 'api/server')
    }
    
externals: [
    resolveExternals
  ],
  
new webpack.ProvidePlugin({
   __extends: 'typescript-extends'
})
  
__dirname: true

function resolveExternals(context, request, callback) {
  return resolveMeteor(request, callback) ||
    callback();
}
 
function resolveMeteor(request, callback) {
  var match = request.match(/^meteor\/(.+)$/);
  var pack = match && match[1];
 
  if (pack) {
    callback(null, 'Package["' + pack + '"]');
    return true;
  }
}

```

Tell TypeScript compiler to include the api dir:

``` 
"include": [
    "src/**/*.ts",
    "api/**/*.ts"
  ],
  "exclude": [
    "node_modules",
    "api/node_modules"
  ],
```

####    (5)npm install --save-dev typescript-extends

####    (6)npm install --save-dev @types/meteor

####    (7)ionic serve

### (二)Chats Page

####    (0) Ionic's component(page) consists 3 files:

-   .html - A view template file written in HTML based on Angular 2's new template engine.
-   .scss - A stylesheet file written in a CSS pre-process language called SASS.
-   .ts - A script file written in Typescript.

####    (1) Remove unused component (page)

1.  ```rm -rf src/pages/home```
2.  remove its declaration in the app module: src/app/app.module.ts, src/app/app.component.ts

####    (2) Create new Component : ChatsPage

1.  src/pages/chats/chats.ts
2.  src/pages/chats/chats.html
3.  src/pages/chats/chats.scss

####    (3) Add new Component (ChatsPage) to NgModule

1.  add ChatsPage to NgModule(app.module.ts);
2.  add ChatsPage to MyApp(app.component.ts)

Add moment module: ```npm install --save mement ```

####    (4) Define models to application

src/models.ts
``` 
export enum MessageType {
  TEXT = <any>'text'
}
 
export interface Chat {
  _id?: string;
  title?: string;
  picture?: string;
  lastMessage?: Message;
}
 
export interface Message {
  _id?: string;
  chatId?: string;
  content?: string;
  createdAt?: Date;
  type?: MessageType
}
```

####    (5) Apply models in application page: ChatsPage

``` 
  <ion-list class="chats">
    <ion-item-sliding *ngFor="let chat of chats | async">
      <button ion-item class="chat">
        <img class="chat-picture" [src]="chat.picture">
        <div class="chat-info">
          <h2 class="chat-title">{{chat.title}}</h2>
 
          <span *ngIf="chat.lastMessage" class="last-message">
            <p *ngIf="chat.lastMessage.type == 'text'" class="last-message-content last-message-content-text">{{chat.lastMessage.content}}</p>
            <span class="last-message-timestamp">{{chat.lastMessage.createdAt }}</span>
          </span>
        </div>
      </button>
      <ion-item-options class="chat-options">
        <button ion-button color="danger" class="option option-remove">Remove</button>
      </ion-item-options>
    </ion-item-sliding>
  </ion-list>
```

####    (6) External Angular 2 Module

Install angular2-moment:
``` npm install --save angular2-moment ```

Import to NgModule
``` 
imports[
MomentModule
]
```
 
Use amCalendar pipe in chats.html
``` 
| amCalendar

``` 

####    (7) Ionic Touch Events

Update chats.html
``` 
<ion-item-options class="chat-options">
  <button ion-button color="danger" class="option option-remove" (click)="removeChat(chat)">Remove</button>
</ion-item-options>
```

Update chats.ts
``` 
  removeChat(chat: Chat): void {
    this.chats = this.chats.map((chatsArray: Chat[]) => {
      const chatIndex = chatsArray.indexOf(chat);
      if (chatIndex !== -1) {
        chatsArray.splice(chatIndex, 1);
      }
 
      return chatsArray;
    });
  }
```

### (三) Meteor Server

####    (1) Create meteor server inside dir ```api```

####    (2) Remove unnecessary folder and files

``` 
rm -rf api/node_modules
rm -rf api/client
rm api/package.json
rm api/package-lock.json
```

####    (3) Make symbolic link to root folder and files

``` 
api$ ln -s ../package.json
api$ ln -s ../package-lock.json
api$ ln -s ../node_modules
```

####    (4)Add TypeScript support to meteor project

``` 
api$ meteor add barbatus:typescript
```

Add server's tsconfig.json,derived from ionic's tsconfig.json.

Link declarations.d.ts from ionic's one.

``` 
api$ ln -s ../src/declarations.d.ts
```

Change file type ext from .js to .ts
``` 
api$ mv server/main.js server/main.ts
```

####    (5) Install babel to meteor project

``` 
api$ npm install --save babel-runtime
api$ npm install --save meteor-node-stubs
```

####    (6) Use MongoDB to persist data

Install meteor-rxjs
``` 
npm install --save meteor-rxjs
```

####    (7) Collections

Create chats collection, api/server/collections/chats.ts

``` 
import { MongoObservable } from 'meteor-rxjs';
import { Chat } from '../models';
 
export const Chats = new MongoObservable.Collection<Chat>('chats');
```

Create messages collection, api/server/collections/messages.ts

``` 
import { MongoObservable } from 'meteor-rxjs';
import { Message } from '../models';
 
export const Messages = new MongoObservable.Collection<Message>('messages');
```

Add a main export file, api/server/collections/index.ts

``` 
export * from 'chats'
export * from 'messages'
```

####    (8)Create data fixtures in the server

Write stubs data in api/server/main.ts

``` 
Meteor.startup(()=>{
if(chats.find({}).cursor.count===0){
    //insert data into chats collection
    //insert data into messages collection
}
})

```

####    (9)Prepare the meteor client

``` 
sudo npm install -g meteor-client-bundler
```

Create a script in packages.json to generate the meteor client bundle:

``` 
"meteor-client:bundle": "meteor-client bundle -s api"
```

Install the ```tmp``` package:
``` 
npm install --save-dev tmp
```

Execute the bundle:

``` 
npm run meteor-client:bundle
```

This will generate a file called ```meteor-client.js``` under the ```node_modules``` dir, which needs to be 
imported in our application like ```import 'meteor-client';```

####    (10)Replace ionic client data with the data fetched from meteor server

