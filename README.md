# go-fcm for FCM HTTP v1 API

[![GoDoc](https://godoc.org/github.com/tevjef/go-fcm?status.svg)](https://godoc.org/github.com/tevjef/go-fcm)
[![Build Status](https://travis-ci.org/tevjef/go-fcm.svg?branch=master)](https://travis-ci.org/tevjef/go-fcm)
[![Go Report Card](https://goreportcard.com/badge/github.com/tevjef/go-fcm)](https://goreportcard.com/report/github.com/tevjef/go-fcm)

Golang client library for [Firebase Cloud Messaging](https://firebase.google.com/docs/cloud-messaging/) v1 API.

The [Legacy FCM HTTP Protocol](https://firebase.google.com/docs/cloud-messaging/http-server-ref) has no construct to make a distinction between Android, iOS and Web notifications. The new [HTTP v1 API](https://firebase.google.com/docs/reference/fcm/rest/v1/projects.messages) does. 

With this library, you:
* Cannot send notifications to multiple registration ids or devices in one request. Use the [Legacy HTTP Protocol](https://firebase.google.com/docs/cloud-messaging/http-server-ref_)

* Cannot receive messages from devices. Use the [Legacy XMPP Protocol](https://firebase.google.com/docs/cloud-messaging/xmpp-server-ref) 

## Getting Started

## CLI App

### Usage

```bash
go install github.com/tevjef/go-fcm/cmd/fcm-send

```

```
NAME:
   go-fcm - Send messages to devices through Firebase Cloud Messaging.

USAGE:
   go-fcm [global options]

VERSION:
   1.0.0

COMMANDS:
     help, h  Shows a list of commands or help for one command

GLOBAL OPTIONS:
   --topic value, -t value       The name of the topic to send a message to.
   --token value, -k value       The device topic or registration id to send a message to.
   --condition value, -c value   The condition to send a message to, e.g. 'foo' in topics && 'bar' in topics
   --title value                 The notification title.
   --body value                  The notification body.
   --validate-only               Validate the message, but don't send it.
   --credentials-location value  Location of the Firebase Admin SDK JSON credentials. [$CREDENTIALS_LOCATION]
   --project-id value            The id of your Firebase project. [$PROJECT_ID]
   --help, -h                    show help
   --version, -v                 print the version
```

## As package

### Usage

```bash
go get github.com/tevjef/go-fcm

```

### Example

```go
import (
	"encoding/json"
	"log"

	"github.com/tevjef/go-fcm"
)

func main() {
	// Create the message to be sent.
	msg := &fcm.SendRequest{
		ValidateOnly: true,
		Message: &fcm.Message{
			Token: "bk3RNwTe3H0:CI2k_HHwgIpoDKCIZvvDMExUdFQ3P1...",
			Notification: &fcm.Notification{
				Title: "FCM Message",
				Body:  "This is a Firebase Cloud Messaging Topic Message!",
			},
			Apns: &fcm.ApnsConfig{
				Payload: &fcm.ApnsPayload{
					Aps: &fcm.ApsDictionary{
						Alert: &fcm.ApnsAlert{
							LaunchImage: "UILaunchImageFileKey",
						},
						Badge:            1,
						Category:         "NEW_MESSAGE_CATEGORY",
						ContentAvailable: int(fcm.ApnsContentAvailable),
					},
				},
			},
			Android: &fcm.AndroidConfig{
				Priority: string(fcm.AndroidHighPriority),
				TTL:      "84000s",
				Notification: &fcm.AndroidNotification{
					Icon:        "ic_notification",
					Color:       "#rrggbb",
					ClickAction: "MainActivity",
				},
			},
		},
	}

	// Create a FCM client to send the message.
	client, err := fcm.NewClient("projectID", "sa.json")
	if err != nil {
		log.Fatalln(err)
	}

	// Send the message and receive the response without retries.
	response, err := client.Send(msg)
	if err != nil {
		log.Fatalln(err)
	}

	log.Printf("%#v\n", response)
}
```
### customer http client Example

```go
    dialer := &net.Dialer{
		Timeout:   8 * time.Second,
		KeepAlive: 60 * time.Second,
	}
	transport := &http.Transport{
		DialContext:         dialer.DialContext,
		MaxConnsPerHost:     200,
		MaxIdleConnsPerHost: 50,
		IdleConnTimeout:     60 * time.Second,
	}
	if proxyUrl != "" {
		proxyUrl, err := url.Parse(proxyUrl)
		if err == nil && proxyUrl != nil {
			transport.Proxy = http.ProxyURL(proxyUrl)
		}
	}
	_ = http2.ConfigureTransport(transport)
	hCli := &http.Client{
		Transport: transport,
		Timeout:   40 * time.Second,
	}
	//fCli, err := fcm.NewClient(conf.FcmProjectID, conf.FcmCert, fcm.WithHTTPClient(hCli))
	fCli, err := fcm.NewClientFromBytes(FcmProjectID, "ca.json", fcm.WithHTTPClient(hCli))
	if err != nil {
		return nil, err
	}
```


### Example JSON sent to FCM HTTP v1 API

```json
{
   "validate_only": true,
   "message": {
     "token": "bk3RNwTe3H0:CI2k_HHwgIpoDKCIZvvDMExUdFQ3P1...",
     or
     "topic": "cats", 
     or
     "condition": "'dogs' in topics || 'cats' in topics", 
     "apns": {
       "headers": {
         "apns-expiration": "14567890",
         "apns-priority": "10",
         "apns-topic": "my-topic",
         "apns-collapse-id": "my-collapse-id"
       },
       "payload": {
         "acme1": "bar",
         "acme2": [
           "bang",
           "whiz"
         ],
         "aps": {
           "alert": {
             "action-loc-key": "PLAY",
             "body": "Acme message received from Johnny Appleseed",
             "launch-image": "UILaunchImageFileKey",
             "loc-args": [
               "Jenna",
               "Frank"
             ],
             "loc-key": "GAME_PLAY_REQUEST_FORMAT",
             "title": "Acme title",
             "title-loc-args": [
               "Jenna",
               "Frank"
             ],
             "title-loc-key": "GAME_PLAY_REQUEST_FORMAT"
           },
           "badge": 1,
           "category": "NEW_MESSAGE_CATEGORY",
           "content-available": 1,
           "sound": "chime.aiff",
           "thread-id": "my-thread-id"
         }
       }
     },
     "android": {
       "collapse_key": "my-collapse-key",
       "priority": "HIGH",
       "ttl": "84000s",
       "restricted_package_name": "com.github.go-fcm",
       "data": {
         "acme1": "bar",
         "acme2": [
           "bang",
           "whiz"
         ]
       },
       "notification": {
         "title": "FCM Message",
         "body": "This is a Firebase Cloud Messaging Topic Message!",
         "icon": "ic_notification",
         "sound": "res_raw_notification_sound.mp3",
         "tag": "my-notification-tag",
         "color": "#rrggbb",
         "click_action": "MainActivity",
         "body_loc_key": "notification_body",
         "body_loc_args": [
           "Jenna",
           "Frank"
         ],
         "title_loc_key": "notification_title",
         "title_loc_args": [
           "Jenna",
           "Frank"
         ]
       }
     },
     "notification": {
       "title": "FCM Message",
       "body": "This is a Firebase Cloud Messaging Topic Message!"
     }
   }
 }
```
