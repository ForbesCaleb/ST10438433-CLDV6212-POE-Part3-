# ST10438433-CLDV6212-POE-Part3-

\# ABCRetailers Web Application \& Azure Functions Backend



This README provides a full, structured explanation of how the \*\*ABCRetailers\*\* system works across both the \*\*ASP.NET Core MVC application\*\* and the \*\*Azure Functions backend\*\*. It covers the architecture, data flow, components, and the purpose of each file.



---



\## 1. System Overview



ABCRetailers is a distributed cloud-based retail system built using:



\* \*\*ASP.NET Core MVC\*\* for the front‑end web application

\* \*\*Azure Functions (Isolated Worker)\*\* for the backend API

\* \*\*Azure Storage Services\*\* for data persistence (Tables, Blobs, Queues)

\* \*\*Azure SQL Database\*\* for authentication and cart storage



The MVC app communicates with the serverless backend through an injected HTTP client (`IFunctionsApi`). Azure Functions handle all business logic, data processing, and interactions with Azure Storage.



---



\## 2. Frontend (ASP.NET Core MVC)



The MVC project is responsible for:



\* User registration \& login

\* Browsing products

\* Managing cart items

\* Placing orders

\* Handling admin actions (CRUD for products, viewing orders)



\### 2.1 Controllers



Each controller serves a specific domain:



\* \*\*HomeController\*\* – Landing page, product list, role-based navigation.

\* \*\*ProductController\*\* – Admin CRUD actions for products.

\* \*\*CustomerController\*\* – Admin/customer details and CRUD.

\* \*\*OrderController\*\* – Order creation and management.

\* \*\*UploadController\*\* – Proof-of-payment uploads.

\* \*\*CartController\*\* – Shopping cart using EF Core.

\* \*\*LoginController\*\* – Authentication using cookies.



All controllers communicate with Azure Functions via `IFunctionsApi`.



\### 2.2 Models \& ViewModels



\* Models represent application data such as `Product`, `Order`, `Customer`, etc.

\* `AuthDbContext` stores users and cart items in \*\*Azure SQL\*\*.

\* ViewModels shape data sent between views and controllers.



\### 2.3 Program.cs (MVC)



Configures the entire web application:



\* Adds MVC services

\* Registers `AuthDbContext`

\* Configures cookie authentication

\* Creates an HttpClient called \*\*Functions\*\*, pointing to your Azure Functions URL

\* Registers the `IFunctionsApi` service



This file wires up dependency injection, routing, and middleware.



---



\## 3. Backend (Azure Functions – Serverless API)



The backend contains multiple Azure Functions grouped by domain.



\### 3.1 Storage Services Used



\* \*\*Azure Table Storage\*\* – Products, Orders, Customers

\* \*\*Azure Blob Storage\*\* – Product images, proof-of-payment files

\* \*\*Azure Queue Storage\*\* – Background order processing



\### 3.2 Functions Overview



Each function file contains operations for a specific domain.



\#### ✔ ProductsFunctions.cs



\* Create product

\* Update product

\* Delete product

\* Get product by ID

\* List all products

\* Uses \*\*Table Storage\*\* \& \*\*Blob Storage\*\*



\#### ✔ CustomersFunctions.cs



\* Create, update, delete customers

\* Get customers

\* Uses \*\*Table Storage\*\*



\#### ✔ OrdersFunctions.cs



\* Create order

\* List orders

\* Get order by ID

\* Enqueues messages for background processing



\#### ✔ QueueProcessorFunctions.cs



\* Triggered when a new order is added to Queue Storage

\* Processes order asynchronously



\#### ✔ BlobFunctions.cs



\* Triggered automatically when a product image is uploaded

\* Supports downstream processing



\#### ✔ UploadsFunctions.cs



\* Accepts proof-of-payment uploads via `multipart/form-data`

\* Saves files to \*\*Blob Storage\*\*



\### 3.3 Helper Classes



\* \*\*TableEntities.cs\*\* – Defines entities for Table Storage.

\* \*\*Map.cs\*\* – Maps between DTOs and Table Storage entities.

\* \*\*HttpJson.cs\*\* – Simplifies JSON handling.

\* \*\*MultipartHelper.cs\*\* – Parses uploaded files.

\* \*\*ApiModels.cs\*\* – DTOs sent between MVC and Functions.



\### 3.4 Program.cs (Functions Backend)



Azure Functions isolated worker setup:



\* Configures worker defaults

\* Registers \*\*TableServiceClient\*\*, \*\*BlobServiceClient\*\*, \*\*QueueServiceClient\*\*

\* Loads Azure Storage connection strings

\* Sets up dependency injection for services



---



\## 4. Data Flow (End-to-End)



Below is the typical flow of actions:



\### \*\*4.1 Viewing Products\*\*



1\. MVC calls `IFunctionsApi.GetProductsAsync()`.

2\. Azure Function queries \*\*Table Storage\*\*.

3\. Returns data to MVC.

4\. MVC displays products on the homepage.



\### \*\*4.2 Adding Product (Admin)\*\*



1\. Admin submits product form.

2\. MVC sends product data to `ProductsFunctions` via `IFunctionsApi`.

3\. Product written to \*\*Table Storage\*\*.

4\. If an image was uploaded, MVC uploads the file to Blob Storage.

5\. `BlobFunctions` may run automatically.



\### \*\*4.3 Customer Registration\*\*



1\. User registers → MVC stores profile in SQL (AuthDbContext).

2\. Also sends profile to Azure Functions → stored in Table Storage.



\### \*\*4.4 Placing an Order\*\*



1\. MVC calls `OrdersFunctions.CreateOrderAsync()`.

2\. Azure Function stores order in \*\*Table Storage\*\*.

3\. Order is added to \*\*Queue Storage\*\*.

4\. `QueueProcessorFunctions` processes it asynchronously.



\### \*\*4.5 Uploading Proof of Payment\*\*



1\. User uploads file.

2\. `UploadController` sends multipart form data.

3\. `UploadsFunctions` stores file in Blob Storage.

4\. File can trigger additional logic.



---



\## 5. Key Architectural Advantages



\* \*\*Serverless scaling\*\* (Functions scale automatically)

\* \*\*Low cost\*\* using Table \& Queue Storage

\* \*\*Separation of concerns\*\* between UI and backend

\* \*\*Reliable message-driven order processing\*\*

\* \*\*Automatic event processing\*\* via triggers (Blob \& Queue)

\* \*\*Secure authentication\*\* using Azure SQL + Cookies



---



\## 6. How Everything Connects



| Component     | Technology        | Purpose                                   |

| ------------- | ----------------- | ----------------------------------------- |

| MVC App       | Azure App Service | Frontend UI, authentication, user actions |

| Functions API | Azure Functions   | All business logic, CRUD operations       |

| Table Storage | NoSQL             | Customers, Products, Orders               |

| Blob Storage  | File Storage      | Images \& proof-of-payment                 |

| Queue Storage | Messaging         | Background order processing               |

| SQL Database  | Relational Data   | Authentication, cart storage              |



Together, these form a complete cloud retail solution.



---



\## 7. Deployment Summary



1\. Deploy MVC app to \*\*Azure App Service\*\*.

2\. Deploy Functions to \*\*Azure Functions\*\* (Isolated worker).

3\. Configure connection strings in Azure App Settings.

4\. Create Storage Account with:



&nbsp;  \* Tables

&nbsp;  \* Queues

&nbsp;  \* Blobs

5\. Create Azure SQL Database.

6\. Run EF Core migrations.

7\. Update MVC base URL for Azure Functions.



---



\## 8. Conclusion



This system is fully cloud-native, scalable, and cost‑efficient.

The MVC frontend provides a user-friendly interface, while the Azure Functions backend handles business logic and integrations with Azure Storage, creating a powerful and flexible retail platform.



If you need improvements, diagrams, or a deployment guide, I can generate them too.

Youtube Link: https://youtu.be/JCuNKKSoyDg

Web URL: https://st10438433-bnhubeceg7d8gmgf.canadacentral-01.azurewebsites.net

