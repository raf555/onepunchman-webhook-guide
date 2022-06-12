# One Punch-Man Webhook

![](https://raw.githubusercontent.com/raf555/onepunchman-webhook-guide/main/images/banner.png)

> One Punch-Man RAW Manga / Webcomic Webhook Notification Service

# Table of contents
1. [Introduction](#pushpin-introduction)
    1. [What is this?](#what-is-this)
    2. [Features](#features)
2. [How to add my URL?](#pushpin-how-to-add-my-url)
    1. [Subscribing](#subscribing)
       1. [For Your Own Application](#for-your-own-application)
       2. [For Your Discord Channel](#for-your-discord-channel)
    2. [Receiving Webhook](#receiving-webhook)
    3. [Securing Webhook Requests](#securing-webhook-requests)
    4. [Example on Node using ExpressJS (with validation)](#example-on-node-using-expressjs-with-validation)
3. [How to Test My URL?](#pushpin-how-to-test-my-url)
4. [How to Remove My URL?](#pushpin-how-to-remove-my-url)
5. [Fail to Receive Webhook Handling](#pushpin-fail-to-receive-webhook-handling)
6. [Download Chapters](#pushpin-download-chapters)
7. [Documentation](#memo-documentation)
    1. [Event Object](#event-object)
    2. [Chapter Object](#chapter-object)
8. [Dev Notes](#dev-notes)

## :pushpin: Introduction

### What is this?

A simple webhook service that notifies your URL whenever One Punch-Man Manga / Webcomic RAW chapter is updated.

### Features

- [x] Simple and easy to use
- [x] Fastest update notification
- [x] Able to send multiple chapters in one request
- [x] Able to detect revision/redraw chapters (manga only)
- [x] Able to retry the notification if it fails to send to your endpoint
- [x] Secured requests
- [x] Discord Webhook support [NEW]
- [x] Download Manga/Webcomic chapters [NEW]

## :pushpin: How to add my URL?

### Subscribing

Assuming you are familiar with webhook, just follow these simple steps:

#### For Your Own Application

1. [IMPORTANT] Your URL should be able to receive POST requests and should reply with a random string which is put in the `opmkey` header everytime our server sends a request within 60 seconds.
2. Open [this](https://opmupdater.herokuapp.com) url.
3. Add your URL.
4. Save the generated key and URL in case you want to [remove](https://opmupdater.herokuapp.com/remove) it later.
5. *Optional*, add secret key to validate request signature, more on this [below](#securing-webhook-requests).
6. Done, your URL should receive the **POST request** containing the event object and chapter object whenever new update arrives.

**IMPORTANT NOTES:**
1. If the server fails to send a request to your endpoint, then the request will be **resent at most 10 times at some interval**.
2. If you provide a **secret key**, please keep it saved somewhere else in your app (such as environment variable). There is no way to retrieve the secret key after subscription.
3. If you want to reset your **subscription key** or **secret key**, you need to remove your URL first and start over the subscription.
4. If you are having difficulties with the implementation, you should see and refer to our example [below](#securing-webhook-requests).
5. [IMPORTANT] If your URL fails to receive the webhook from our server more than 10 times in a month, your URL will be removed and you should start over the subscription.

#### For Your Discord Channel

1. Create Webhook in your Discord Server, please refer to [Discord's tutorial](https://support.discord.com/hc/en-us/articles/228383668) for more information.
2. Go to the [form](https://opmupdater.herokuapp.com/discord).
3. Fill the form with previously created Discord Webhook URL, fill all required field and hit subscribe button.
4. Save the generated key and URL in case you want to [remove](https://opmupdater.herokuapp.com/remove) it later.
5. Congratulations! Your channel will be notified whenever new chapter arrives.
6. You are pretty much done, you can continue to [test](#pushpin-how-to-test-my-endpoint) or [remove](#pushpin-how-to-remove-my-endpoint) sections. 

**NOTES:**
1. You can fill more than 1 role id.
2. Chapter will be sent as an embed messages. Here are the examples.
    1. [Manga](https://i.ibb.co/tKKpnB2/image.png)
    2. [Webcomic](https://i.ibb.co/cYTsXJR/image.png)

**Section below is for your own application, if you subscribed a discord webhook URL, you can ignore this section.**

### Receiving Webhook

When the system finds a new chapter, it will fire POST request immediately to your URL. The content of the notification are:

1. Headers
    1. `opmkey`: Random string unique for each Webhook Event. You should reply with this string for every requests
    2. `opm-signature-sha256`: HMAC SHA256 signature of the request body (signed with the secret value provided when subscribing)
2. Body
    1. `Event Object`: Object with event **update** containing **List of Chapter Object**. More on this on [documentation](#memo-documentation).
        - if your endpoint URL is Discord Webhook URL, this will be a Discord Webhook [Message Object](https://discord.com/developers/docs/resources/webhook#execute-webhook) instead.

### Securing Webhook Requests

Securing webhook notification request is necessary to avoid your URL getting spoofed by unknown requests. You can use proved method to prove that the requests are coming from our server.

#### Signature Validation

If you provide secret key when subscribing, our server will provide a `opm-signature-sha256` header containing a signed request body using HMAC SHA256 with the provided secret key. More on how to do this is (kinda) explained in the example below.

### Example on Node using ExpressJS (with validation)

Example below implements the previously explained validation. The validation are implemented in **validate** function as a middleware.

You can use this example as starter template.

Please refer to [this repo](https://github.com/raf555/onepunchman-webhook-client)
or go ahead to [Documentation Page](https://opmupdater.herokuapp.com/documentation)

## :pushpin: How to Test My URL?

After registration completes, you can test your URL to see if it can receive the webhook correctly.

When you subscribed an URL, the website should save the subscription key for you in your browser.

You can go to [Dashboard Page](https://opmupdater.herokuapp.com/dashboard) to test your URL, there should be three test options.

![](https://i.ibb.co/ncw0pDG/image.png)

1. Will send a POST request containing [Event Object](#event-object) with **test** event to your URL.
    - If your URL is a Discord Webhook URL, this will send a GET request instead.

2. Will send a POST request containing [Event Object](#event-object) with **update** event with latest **Manga** chapter to your URL.
    - If your URL is a Discord Webhook URL, this will send a Discord Webhook [Message Object](https://discord.com/developers/docs/resources/webhook#execute-webhook) with embeds instead.

3. Will send a POST request containing [Event Object](#event-object) with **update** event with latest **Webcomic** chapter to your URL.
    - If your URL is a Discord Webhook URL, this will send a Discord Webhook [Message Object](https://discord.com/developers/docs/resources/webhook#execute-webhook) with embeds instead.


## :pushpin: How to Remove My URL?

You can go to [Dashboard Page](https://opmupdater.herokuapp.com/dashboard) to remove your URL.

## :pushpin: Fail to Receive Webhook Handling

Our server will resend the Webhook Event at most 10 tries at some interval. If your URL returns a 200 code with invalid response (it should be a random string in the `opmkey` header), the request will be considered as a fail request. **Please keep in mind if your URL fails to receive the Webhook more than 10 times in a month, your URL will be removed**.

## :pushpin: Download Chapters

You can go to [Download Page](https://opmupdater.herokuapp.com/download) to download Manga/Webcomic Chapters.

# :memo: Documentation

## `Event Object`

Your endpoint will receive event object when first time added to system and receiving new updates.

#### Object

- **type** `String`: Object type (**test** / update).
- **source** `String`: Update source (**null** / manga / webcomic).
- **chapters** `List of Chapter Object`: List of new chapter based on the source in ascending order.

_note: type "test" and source "null" are only sent when first added to the system_

```javascript
{ 
  "type": (String),
  "source": (String),
  "chapters": (List of Chapter Object)
}
```

#### Example

```javascript
{
  "type": "test",
  "source": null,
  "chapters": []
}
```

#### New Chapters Example

```javascript
{
 "type": "update",
 "source": "manga",
 "chapters": [
  {
   "id": "3269754496479987879",
   "chapter": "153",
   "revision": true,
   "redraw": true,
   "thumbnail": "https://some.image/url.jpg",
   "timestamp": 1630602000000,
   "url": "https://tonarinoyj.jp/episode/3269754496479987879",
   "totalPages": 17,
   "downloadURL": "https://opmupdater.herokuapp.com/download#/manga/3269754496479987879"
  }
 ]
}
```

## `Chapter Object`

Containing data from new chapter.

#### Object

- **id** `String` : Chapter id based on the source.
- **chapter** `String`: Chapter number based on the source (TonariNYJ/ONE's Site).
- **revision** `Boolean`: Indicates chapter revision.
- **redraw** `Boolean`: An alias for **revision** field.
- **thumbnail** `String`: Chapter thumbnail URL.
- **timestamp** `Long`: Chapter timestamp.
- **url** `String`: Chapter URL.
- **totalPages** `Number`: Chapter total pages.
- **downloadURL** `String`: Chapter download URL.

```javascript
{
  "id": (String),
  "chapter": (String),
  "revision": (Boolean),
  "redraw": (Boolean),
  "thumbnail": (String),
  "timestamp": (Long),
  "url": (String),
  "totalPages": (Number),
  "downloadURL": (String)
}
```

#### Example

```javascript
{
 "type": "update",
 "source": "manga",
 "chapters": [
  {
   "id": "3269754496479987879",
   "chapter": "153",
   "revision": true,
   "redraw": true,
   "thumbnail": "https://cdn-scissors.gigaviewer.com/image/scale/41085ae138d61ce997a5a2ec37b571f3100c2191/enlarge=0;height=450;no_unsharpmask=1;quality=90;version=1;width=320/https%3A%2F%2Fcdn-img.tonarinoyj.jp%2Fpublic%2Fepisode-thumbnail%2F3269754496479987879-24fb75cee700fb5ea50c4465b5b3e0f4%3F1630667893",
   "timestamp": 1630602000000,
   "url": "https://tonarinoyj.jp/episode/3269754496479987879",
   "totalPages": 17,
   "downloadURL": "https://opmupdater.herokuapp.com/download#/manga/3269754496479987879"
  }
 ]
}
```
