## lab-nine - Designing and Implementing a Microservices Architecture

### Objectives:

* Design a microservices architecture based on a given set of requirements.
* Simulate the decomposition of a monolithic architecture into microservices.
* Reflect on the design decisions and discuss alternative approaches.


### Tasks:

* Application Brief: Participants are given a description of a monolithic e-commerce application handling user management, product catalog, order processing, and customer support.

* Task: Identify and outline potential microservices based on domain-driven design principles. Participants will determine service boundaries, define how services will communicate, and plan how to handle shared data.
 
* Migration Roadmap: Develop a detailed plan for migrating the identified monolithic components to microservices. This plan should include prioritization of services to be migrated, identification of dependencies, and a strategy for data migration.

* Architecture Documentation: Document the proposed microservices architecture, illustrating the interaction between services and the migration steps. Include a brief narrative explaining the rationale behind key decisions.




# Solution

Assuming that the challenge to migrate monolithic app concerns only backend and front end is already was migrated

## General Approach

* Strangler Fig Pattern will be the  approach for this migration from monolithic platform to microservices
* Analyze costs about Cloud provider, Mongo cluster, integrations, team members
* The microservices will be allocated in Cloud provider AWS to provide auto scaling, auto management serverless function, retry and resilience solutions
* Database selected will be Mongo DB (no sql database allow to decouple domains in a cluster)
* Use an API gateway and loader balance to redirect requests 
* Use cache memory for frequent queries 
* Use CQRS in some microservices to consider Write and Read operations
* Use feign client to integrate calls from remaining monolithic service to microservices migrated
* Use event-driven (Kinesis, lambda) solutions for async hard processing
* Each migrated service will pass for a QA automation and manual test and Security audit


## Roadmap

### 1. Decomposition

### 1.1 Identifying Service Boundaries

####  user-management-service
	
 It will allow manage the following actions: 
 	- signup
 	- login
 	- security profile/roles

 Justification: We need to consider an isolated module to manage the flows related to user account registration and login. Thinking in a modularity and scalability environment  

#### product-service

 It will allow manage the following actions: 
	- create a product that includes name, price, desc, photo
	- update a product (name, price, desc, photo)
	- delete a product by Id
	- retrieve a product by name / id

Justification: it is necessary a specialized module to handle products. That allow us handle a set operations about it. Thinking in a modularity and scalability environment  

#### order-service
  
  It will allow manage the following actions:

  - Create an order that contains order number, amount, product items, createdAt, status
  - Cancel order
  - Retrieve/see orders

  Justification: Modiule specialized in order and its processing. It will have a hard processing tasks and transactions. Thinking in a modularity and scalability environment  

#### customer-support-service

  It will allow manage the following actions:

  - Create a dispute
  - Call with asesor 
  	- chat
  	- call
  - Retrieve / watch my disputes

  Justification: Module allos resolve disputes and questions from users of the e-commerce platform about account or their orders. Thinking in a modularity and scalability environment  


### 1.2 Domains - Ubiqous language

- User: 
	- user/client:  person that makes a register and capture its personal and payment information in the  e-commerce platform
	- Account info: email address, full name, phone, bankging card info, address from the user/client
	- Signup : Process where person create an account in the ecommerce platform
	- Login : Process where person authenticate in the e-commerce platform to do different actions related to the products (search, add to car, pay)

- Order:
	- Order: Entity that allows add products and pay for them. It generate and order number, date creation, payment details, show the status 
	- Status: State of the order can be CREATED, PENDING, CONFIRMED, IN_TRANSIT, DELIVERED, CANCELLED

- Product
	- Product: Item that is published in the e-commerce platform , has price, photo, description, vendor, product status
	- product status: descrive the status such as active / not active
	- vendor: provider who sell the product

- Customer support
	- customer support: Area that give attention and solution for customer disputes about orders or account info
	- Channel: Define the via through customer/user contact to support assistant can be Chatbox, chat with assistant, phone call

### 2. Identify Dependencies

* User management (Authentication, Registration) is the lower coupled functionalities else others
* Products - Orders are high coupled in monolithic platform
* Customer support has integration with Chatbot
* Database is only one and contains data for all modules
* Some processes taking a long time period and are processed using spring batch (it considered to decoupled in event driven flows) 

### 3. Identify which microservice migrate

In the first scope called Q1
User management will be the first in migrate considering the Strangler Fig Pattern
* The feature Is lower coupled than other functionalites.
* We can separate this small feature including their design database
* We can add and integrate Security layer using AWS Cognito for authentication

In the follow periods could be migrating the follow functionalities

Q2: Products
Q3: Orders
Q4: Customer support

### 4. Data migration

Q1.

Work with devops team, define, analyse costs and create cluster MongoDB

user-management-service

- Define and create database  user-db
- Define and create collections user, user-cognito

For the follow periods, do the same for remaining functionalities

Q2. product-service

Q3. order-service

Q4. customer-support-service

### 5. System Integrity:

* Pilot period for tests
* Integration and performance tests

### 6. Maintaining Data Consistency
* In migration ensure data
* Event driven service should have retry, ttl for stream saved, dead-letter topics, logging

### 7. Service Discovery and Fault Tolerance
Implement AWS solutions like API Gateway, Zuul server, load balancer, and Resilience patterns in cloud or frameworks in code

## Architecture

![img.png](img.png)