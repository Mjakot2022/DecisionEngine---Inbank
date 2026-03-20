# DecisionEngine---Inbank

## Overview
A decision engine that determines the maximum approvable loan amount for a given person, built with a Java Spring Boot backend and a plain HTML/JavaScript frontend.

## How to Run
1. Clone the repository
2. Open the project in IntelliJ IDEA
3. Run `LoanApplication.java`
4. Open `http://localhost:8080` in your browser

## Design Choices

### Backend — Java with Spring Boot
Java was chosen as the primary language as it is Inbank's core backend technology. Spring Boot was selected over plain Java Servlets because it eliminates boilerplate code around HTTP request handling. 

In traditional Java Servlets, handling an HTTP request requires manually extending `HttpServlet`, parsing the request body, and writing the response manually. Spring Boot abstracts all of this through annotations:
- `@RestController` — marks a class as an HTTP request handler and automatically converts return values to JSON
- `@PostMapping` — maps a method to a specific HTTP endpoint
- `@RequestBody` — automatically deserializes incoming JSON into a Java object using the Jackson library under the hood

This allowed the focus to remain on business logic rather than infrastructure.

### Single API Endpoint
The assignment requires a single endpoint:
```
POST /loan/decision_engine
```
which accepts a JSON body with `personalCode`, `loanAmount`, and `loanPeriod`, and returns a decision and approved amount.

POST was chosen over GET because the request sends data for processing rather than fetching existing data, and POST allows a structured JSON body rather than passing values through the URL.

### Data Transfer Objects (DTOs)
Two simple classes were created to represent incoming and outgoing data:
- `LoanRequest` — maps incoming JSON fields to Java fields. Requires setters because Spring fills it in automatically by matching JSON field names to setter names.
- `LoanDecision` — represents the response. Uses a constructor because it is created manually in the business logic and only needs to be read by Spring when converting to JSON.

### Decision Engine Logic
The core algorithm is based on the credit score formula provided:
```
credit score = (creditModifier / loanAmount) * loanPeriod
```
Rather than checking the credit score for the requested amount, the engine solves for the maximum approvable amount directly:
```
loanAmount = creditModifier * loanPeriod
```
This is because the task requires always returning the maximum approvable amount regardless of what was requested. The requested loan amount does not influence the output — the engine always seeks the best possible outcome for the applicant.

If the maximum approvable amount for the requested period falls below the 2000€ minimum, the engine extends the loan period incrementally up to 60 months until a valid amount is found.

The credit modifier is hardcoded per personal code as a mock for what would in production be an external registry lookup:
- `49002010965` — debt, no approval
- `49002010976` — segment 1, modifier 100
- `49002010987` — segment 2, modifier 300
- `49002010998` — segment 3, modifier 1000

### Input Validation
The engine validates that loan amount is between 2000€ and 10000€ and loan period is between 12 and 60 months before processing. Invalid inputs return a NEGATIVE decision immediately.

### Frontend — Plain HTML/CSS/JavaScript
A plain HTML page was chosen over React or TypeScript to keep the project simple and focused on the backend logic. JavaScript's `fetch` API sends a POST request to the backend and displays the result. No framework or build tool is needed — Spring Boot serves the `index.html` file automatically from the `src/main/resources/static/` directory.

## What I Would Improve
The requested loan amount is effectively redundant in the current algorithm — the engine ignores it and always calculates the maximum approvable amount anyway. I would redesign the algorithm to make the requested amount meaningful, for example by first checking whether the exact requested amount can be approved, and only searching for alternatives if it cannot. This would make the engine more realistic and the input more purposeful.
