# System Architecture

## Mobile Currency Exchange System

```mermaid
flowchart TB

    NBP["NBP API<br/>National Bank of Poland<br/><br/>Official Exchange Rates<br/>Data Owner: NBP"]

    BACKEND["Backend Web Service<br/>Spring Boot REST API<br/><br/>Business Logic<br/>Validation<br/>Authorization<br/>NBP Integration"]

    DB[("PostgreSQL Database<br/><br/>Users<br/>Transactions<br/>Wallet Balances<br/><br/>Data Owner: Our Service")]

    WIRELESS["WIRELESS SEGMENT<br/>Wi-Fi / 4G / 5G"]

    APP["Android Mobile App<br/>Kotlin<br/><br/>Registration / Login<br/>Exchange Rates<br/>Buy / Sell<br/>Wallet<br/>Transaction History"]

    USER["User"]

    USER --> APP
    APP -->|"HTTPS / REST Request"| WIRELESS
    WIRELESS -->|"HTTPS / REST"| BACKEND
    BACKEND -->|"Read / Write"| DB
    BACKEND -->|"HTTPS<br/>Exchange-rate request"| NBP
