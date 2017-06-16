## Why?
The new API aims to serve both client and server-side rendering by removing any dependencies on .NET from the output HTML and having both consume JSON and run in JavaScript. This means that we have removed the usage of Views, View Models and Controllers in our MVC application to make it exclusively an API application. The goal of this philosophy is to significantly reduce development and maintenance effort, while simultaneously being able to improve front-end performance by taking advantage of the latest front-end technologies on the [NodeJS](https://www.nodejs.org) stack.

## Architecture
The architecture provides a distinct line of separation between the APIs and the frontend framework. Previously, through ASP.NET MVC the controllers & view models were coupled with the business logic that was used to build the views. By removing these these from our .NET application, our .NET application is only restricted to business logic and will send all responses in JSON.
The API's response can then be consumed by a client-side application (such as a single-page app) or a server-side NodeJS application without knowledge of the internal workings of the API - **black box concept**. Additionally, if the application being developed supports both server & client-side interactions, then it is able to share the same modules between both applications - **isomorphic JavaScript**.

![alt text](https://github.com/IncredibleWeb/architecture/blob/master/Img/Incredible-Api-Architecture.jpg)

## Chapters
- **[API](https://github.com/IncredibleWeb/architecture/blob/master/Api.md)**
