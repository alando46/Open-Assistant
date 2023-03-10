# Inference

### From the perspective of the Text Client

```mermaid
---
title: Text Client Workflow
---
graph LR;
    subgraph OA Server
    /auth/login
    id1["/chats/{chat_id}/messages"]
    id2["{backend_url}/chats/{chat_id}/messages/{message_id}/events"]
    end

    subgraph Text_Client
    id3["SSEClient(response)"]
    end

    Text_Client == 1. GET ==> /auth/login;
    /auth/login -- bearer_token, chat_id--> Text_Client;

    Text_Client == 2. requests prompt ==> user;
    user == submits prompt ==> Text_Client;

    Text_Client == 3. POST ==> id1["/chats/{chat_id}/messages"];
    id1["/chats/{chat_id}/messages"] -- assistant_message_id --> Text_Client;

    Text_Client == 4. GET ==> id2["{backend_url}/chats/{chat_id}/messages/{message_id}/events"];
    id2["{backend_url}/chats/{chat_id}/messages/{message_id}/events"] -- assistant_message_id --> id3["SSEClient(response)"];

    linkStyle 0 stroke-width:2px,fill:none,stroke:green;
    linkStyle 1 stroke-width:2px,fill:none,stroke:green;
    linkStyle 2 stroke-width:2px,fill:none,stroke:purple;
    linkStyle 3 stroke-width:2px,fill:none,stroke:purple;
    linkStyle 4 stroke-width:2px,fill:none,stroke:blue;
    linkStyle 5 stroke-width:2px,fill:none,stroke:blue;
    linkStyle 6 stroke-width:2px,fill:none,stroke:grey;
    linkStyle 7 stroke-width:2px,fill:none,stroke:grey;
```

For development, a basic REPL client is used as a chat interface, built around [Typer](https://typer.tiangolo.com/).

The client authenticates with the backend open assistant server, which provides a chat id for the client.

The client then collects the user prompt. 

the client posts to the endpoint /chats/{chat_id}/messages
included in this request is the message content and a parent_id (assistant's response to prior message)

in the response, the server will return an assistant_message id. 

The client will use this id to make a GET request to 
{backend_url}/chats/{chat_id}/messages/{message_id}/events
critically, in this GET request the client passes stream=True in the body and "Accept": "text/event-stream" in the headers.

The response is then used to instantiate and SSEClient, which - via its events() method,
returns an iterable that can be used to print out inference results, one token at a time.

After exhausting the events iterable (ie inference is complete), the 


### From the perspective of the OA Inference Server