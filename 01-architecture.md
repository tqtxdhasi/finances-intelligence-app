# 01 – System Architecture

## Overview

The Receipt Intelligence App follows a **cloud‑first** architecture. The mobile client communicates with a backend API that handles all data persistence, file storage, and business logic. No local database is used in V1.

## High‑Level Diagram

```mermaid
graph TD
    A[Mobile App<br/>React Native] -->|HTTPS| B[Backend API<br/>Node.js + Express]
    B --> C[PostgreSQL]
    B --> D[File Storage<br/>S3/R2]
    B --> E[Currency API<br/>exchangerate.host]
    C --> F[(receipts, items,<br/>products, categories)]
    D --> G[Images/PDFs]
