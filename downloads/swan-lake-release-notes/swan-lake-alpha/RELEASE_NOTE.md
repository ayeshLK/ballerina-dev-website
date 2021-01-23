### Standard Library

#### WebSub Module Improvements

##### Hub Related Changes

- Default implementation for `websub:Hub` has been removed from the module.
- API specification for **WebSub Hub** is moved to `websubhub` module.

##### Publisher Related Changes

- **WebSub Publisher** related implementation is moved to `websubhub` module.

##### WebSub Subscriber Related Changes

- Two new configurations are introduced to `@websub:SubscriberServiceConfiguration`
  - accept - The expected media type
  - acceptLanguage - The expected language type
- API specification for `@websub:SubscriberService`
  - `onIntentVerification` and `onNotification` functions are removed.
  - `onSubscriptionVerification`, `onEventNotification`, `onSubscriptionValidationDenied` functions are introduced.
- Updated `websub:SubscriberService` sample implementation is as follows.

```ballerina
    listener websub:Listener testListener = new(9090);

    @websub:SubscriberServiceConfig {
        path: "/websub",
        subscribeOnStartUp: false,
        target: ["http://localhost:9191/websub/hub", "http://websubpubtopic.com"],
        leaseSeconds: 36000,
        secret: "Kslk30SNF2AChs2"
    }
    service /subscriber on testListener {
        remote function onSubscriptionValidationDenied(SubscriptionDeniedError msg) 
                            returns Acknowledgement {
            // execute subscription validation denied action
        }

        remote function onSubscriptionVerification(SubscriptionVerification msg)
                            returns SubscriptionVerificationSuccess|SubscriptionVerificationError {
            // execute subscription verification action
        }

        remote function onEventNotification(ContentDistributionMessage event) {
            // execute event notification received action
        }
    }
```

#### Introduced Modules

##### WebSubHub Module

- This module contains the API specifications and implementations related to **WebSub Hub**, **WebSub Hub Client** and **WebSub Publisher**.
- This is an inter-dependent module for `websub` module.

###### Hub Implementation

- Default `websub:Hub` implementation has been removed and Language specific API abstraction is defined at `websubhub:Service`.
- Updated API specification compliant **WebSub Hub** sample implementation is as follows.

```ballerina
    listener websubhub:Listener hubListener = new(9001);

    service /websubhub on hubListener {

        remote function onRegisterTopic(TopicRegistration message)
                            returns TopicRegistrationSuccess|TopicRegistrationError {
            // implement action to execute on topic-registration
        }

        remote function onDeregisterTopic(TopicDeregistration message)
                            returns TopicDeregistrationSuccess|TopicDeregistrationError {
            // implement action to execute on topic-deregistration
        }

        remote function onUpdateMessage(UpdateMessage msg)
                            returns Acknowledgement|UpdateMessageError {
            // implement action to execute on content-update for topic
        }
    
        remote function onSubscription(Subscription msg)
                            returns SubscriptionAccepted
                                    |SubscriptionPermanentRedirect
                                    |SubscriptionTemporaryRedirect
                                    |BadSubscriptionError
                                    |InternalSubscriptionError {
            // implement action to execute on new subscription
        }

        remote function onSubscriptionValidation(Subscription msg)
                            returns SubscriptionDeniedError? {
            // implement action to execute on subscription validation
        }

        remote function onSubscriptionIntentVerified(VerifiedSubscription msg) {
            // implement action to execute on subscription intent verification
        }

        remote function onUnsubscription(Unsubscription msg)
                            returns UnsubscriptionAccepted
                                    |BadUnsubscriptionError
                                    |InternalUnsubscriptionError {
            // implement action to execute on unsubscription
        }

        remote function onUnsubscriptionValidation(Unsubscription msg)
                            returns UnsubscriptionDeniedError? {
            // implement action to execute on unsubscription validation
        }

        remote function onUnsubscriptionIntentVerified(VerifiedUnsubscription msg){
            // implement action to execute on unsubscription intent verification
        }
    }
```

###### Hub Client Implementation

- `websubhub:Client` is introduced to distribute updated content to subscribers.
- Following is a sample use-case for **WebSub Hub Client**.

```ballerina
    service /websubhub on hubListener {
        remote function onSubscriptionIntentVerified(websubhub:VerifiedSubscription msg) {

            // you can pass client config if you want 
            // say maybe retry config
            websub:HubClient hubclient = new(msg);
            check start notifySubscriber(hubclient);
        }

        function notifySubscriber(websubhub:HubClient hubclient) returns error? {
            while (true) {
                // fetch the message from MB
                check hubclient->notifyContentDistribution({
                    content: "This is sample content delivery"
                });
            }   
        }
    }
```

###### Publisher Implementation

- `websub:PublisherClient` is moved to `websubhub:PublisherClient`.
