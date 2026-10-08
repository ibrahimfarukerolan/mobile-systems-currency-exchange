Mobile Currency Exchange System
1. Project Overview

The project is a mobile currency-exchange system consisting of an Android mobile application, a web service, and a database.

The application allows users to:

register and log in;
view current exchange rates;
view historical exchange rates;
simulate account funding;
buy and sell currencies;
view transaction history;
view wallet balances.

Exchange-rate data is obtained from the National Bank of Poland (NBP) API.

All account funding is simulated. The application does not process real financial transactions or real money.

The backend service provides the business logic, API integration, validation, authorization, and communication with the database. Data-changing operations are validated and their authoritative outcomes are recorded by the service.

2. Team

The project is developed by a two-person team.

Team member 1: [Name]
Team member 2: [Name]
3. Technology Stack
Mobile Application
Android
Kotlin
Android Studio
Backend Service
Java
Spring Boot
REST API
Database
PostgreSQL
Relational database
External Data Source
National Bank of Poland (NBP) API
Exchange-rate data is obtained from the NBP service.
Communication
HTTP/HTTPS
REST-based communication between the mobile application and backend service
Version Control
Git
GitHub
Authentication and Authorization
User authentication through the backend service
Token-based authorization will be considered for authenticated requests
4. System Architecture

The system consists of three main application components:

Android mobile application
Backend web service
PostgreSQL database

The backend also communicates with the external National Bank of Poland API to obtain exchange-rate information.

Request Path

A typical request follows this path:

User → Android Mobile App → Wi-Fi / 4G / 5G → Backend REST API → Database

For exchange-rate information, the backend may additionally communicate with:

Backend REST API → NBP API

The wireless segment represents the network connection between the mobile application and the backend service.

5. Components and Responsibilities
Android Mobile Application

The mobile application provides the user interface and allows users to:

register and log in;
view exchange rates;
view historical rates;
fund their account using simulated funds;
buy and sell currencies;
view wallet balances;
view transaction history.
Backend Web Service

The backend is responsible for:

authentication and authorization;
business logic;
validation of data-changing operations;
processing currency buy and sell operations;
managing wallet balances;
recording transactions;
communicating with the NBP API;
communicating with the database.

The backend service records the authoritative outcome of data-changing operations.

PostgreSQL Database

The database stores application data such as:

users;
transactions;
currency-wallet balances.

Additional tables may be introduced during later development if required.

National Bank of Poland API

The NBP API is the external source of official exchange-rate data used by the application.

NBP is the owner of the external exchange-rate data.

6. Data Ownership
Data	Owner
Official exchange rates	National Bank of Poland
User account information	Our application/service
Wallet balances	Our application/service
Transactions	Our application/service
User preferences	User/application

The application does not modify the official exchange-rate data provided by NBP.

7. Normal User Journeys
Journey 1 — View Exchange Rate
User opens the mobile application.
User selects a currency pair, for example EUR/PLN.
The mobile application sends a request through the wireless network.
The backend processes the request.
The backend obtains the required exchange-rate data.
The exchange rate is returned to the mobile application.
The user sees the exchange rate.
Journey 2 — Buy Currency
User logs in.
User selects a currency to buy.
User enters the amount.
The mobile application sends the transaction request to the backend.
The backend validates the request and user balance.
The backend processes the transaction.
The transaction and updated wallet balance are recorded in the database.
The backend returns the authoritative transaction result.
The mobile application displays the updated wallet balance and transaction result.
8. Connectivity-Loss Journeys
Journey 1 — Connection Lost Before Response
User requests an exchange rate.
The mobile application sends the request.
The wireless connection is interrupted.
The response does not reach the mobile application.
The application cannot determine whether the request was processed successfully.
The result is therefore treated as uncertain rather than automatically assuming success.
The application informs the user that the result could not be confirmed.
Journey 2 — Stale Exchange-Rate Data
User opens the application while the network is unavailable.
The application has previously stored exchange-rate data.
The application displays the previously available rate.
The data may be stale because it was obtained earlier.
The application shows the time/date of the last successful update.
When connectivity is restored, the application can request fresh data.
9. Initial Risk List
Risk	Description	Initial Mitigation
Network connection loss	The mobile application may lose connectivity while communicating with the backend.	Detect connection failures and provide appropriate feedback.
Stale data	Previously retrieved exchange rates may no longer represent the current NBP rates.	Store and display the timestamp of the last successful update.
Lost response	A request may be sent successfully but its response may be lost.	Treat the outcome as uncertain until it can be confirmed.
NBP API unavailable	The external exchange-rate service may be temporarily unavailable.	Handle the error and use previously available data when appropriate.
Slow network	A request may take too long to complete.	Use request timeouts and inform the user.
Database failure	The backend may temporarily be unable to access the database.	Handle database errors without reporting a false successful transaction.
Invalid data	External or user-provided data may be invalid or unexpected.	Validate data before processing or storing it.
Unauthorized access	A user may attempt to access another user's data or perform unauthorized actions.	Implement authentication and authorization on the backend.
10. Project Boundary

The project will implement an integrated:

Mobile Application + Web Service + Database

The source code will be maintained in Git throughout the semester.

The project will provide reproducible instructions for:

starting the mobile application;
starting the backend service;
starting the database;
configuring required services;
demonstrating the main application functionality.

