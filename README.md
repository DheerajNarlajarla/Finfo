# finfo

This is the README file for the finfo project. Edit this file to add project details, setup instructions, and usage information.

If this following were to be the primary project description and you should shift your focus to this

### **Project Title: "Finfo" - A Modular Financial Data Acquisition Platform**

We can unofficially call our project "Janus," after the Greek god of beginnings, gates, and transitions, reflecting its role as a gateway to financial data and its two-faced nature of looking at both user commands and external data sources.

---

### **1. High-Level Mission Statement**

Project Janus is a containerized, microservice-based web application designed to act as a foundational prototype for a sophisticated financial data acquisition and processing system. At its core, it allows a user to issue commands through a simple web interface. These commands are then interpreted and executed by a chain of independent backend services, which perform automated web scraping of financial data. The gathered data is then stored in a flexible NoSQL database for future retrieval and analysis.

The architecture is explicitly designed for modularity and scalability, allowing for the future integration of more advanced AI, additional data sources, and complex analysis tools without requiring a complete system overhaul.

---

### **2. Core Architectural Philosophy: The Microservice Approach**

The entire infrastructure is built upon a **microservice architecture**, orchestrated by **Docker and Docker Compose**. Instead of a single, monolithic application, Janus is composed of five distinct, independent services, each running in its own Docker container.

This approach provides several key technical advantages:
* **Separation of Concerns:** Each service has a single, well-defined responsibility. The web server only handles web traffic, the application server only handles user logic, the database only handles storage, and the agents only handle their specific tasks. This makes the system easier to develop, debug, and understand.
* **Technological Flexibility:** Each microservice can, in theory, be written in a different programming language best suited for its task. For our prototype, we will use Python across the board for consistency.
* **Scalability:** If our Data Miner agent becomes a bottleneck, we can scale it by deploying multiple instances of just that container, without touching any other part of the system.
* **Resilience:** A failure in one non-critical service (e.g., the Data Miner) won't necessarily bring down the entire application. The web interface could remain responsive and report the failure.

---

### **3. Detailed Component Breakdown (The 5 Containers)**

#### **a. Container 1: The Apache Reverse Proxy (The Gatekeeper)**
* **Technology:** Apache HTTP Server 2.4
* **Role:** This container is the sole entry point for all traffic from the outside world (i.e., your web browser). It is configured to act as a **reverse proxy**.
* **Technical Details:**
    * It listens on standard ports (e.g., port 80 for HTTP).
    * Its primary function is to forward incoming requests to the appropriate internal service. All traffic intended for the web application will be proxied to the Django container on its internal port.
    * **Security:** It acts as a robust, security-hardened shield, protecting our application containers (which are not directly exposed to the internet) from a wide range of common web attacks.
    * **Performance:** It is highly optimized for handling high volumes of connections and can be configured to serve static assets (CSS, JavaScript, images) directly, taking that load off our Python-based Django application and significantly speeding up page load times.

#### **b. Container 2: The Django Web Application (The Command Center)**
* **Technology:** Python 3.x, Django 4.x
* **Role:** This is the user-facing part of the application and the primary orchestrator of user-initiated tasks.
* **Technical Details:**
    * **User Interface (UI):** It serves the HTML templates that constitute the web pages a user interacts with, including the input form for submitting commands.
    * **Request Handling:** It processes incoming HTTP requests from the reverse proxy, routing them to the appropriate view logic.
    * **Backend Client:** When a user submits a task, the Django view does *not* execute the task itself. Instead, it acts as a client, making an internal HTTP API call to the **Smart Router Agent**. This asynchronous-style delegation is crucial for keeping the web interface responsive.
    * **Data Presentation:** It contains the logic to query the MongoDB database directly to fetch results and present them to the user in a readable format on a results page.

#### **c. Container 3: The Smart Router Agent (The Dispatcher)**
* **Technology:** Python 3.x, FastAPI or Flask (lightweight API frameworks).
* **Role:** This service acts as the central "brain" or dispatcher for our backend. It decouples the main Django application from the specific "worker" agents.
* **Technical Details:**
    * It runs a simple web server that exposes an internal REST API (e.g., an endpoint at `POST /api/v1/submit-task`).
    * It receives a task description from the Django container (e.g., a JSON payload like `{"task_type": "scrape", "target": "some-website.com"}`).
    * In our prototype, its logic will be straightforward: upon receiving a task, it will immediately delegate it by making another internal API call to the Data Miner Agent.
    * In the future, this is where more complex logic would reside, allowing it to interpret the task and route it to one of many different worker agents.

#### **d. Container 4: The Data Miner Agent (The Worker)**
* **Technology:** Python 3.x, FastAPI/Flask, and scraping libraries like **BeautifulSoup4**, **Scrapy**, or **Playwright**.
* **Role:** This is the workhorse of the system. It performs the heavy lifting of actually acquiring the data.
* **Technical Details:**
    * Like the router, it exposes a simple internal API (e.g., `POST /api/v1/execute-scrape`).
    * It receives a very specific, actionable instruction from the Smart Router.
    * It uses libraries like `requests` to fetch web page content or more advanced tools like `Playwright` to control a headless browser for scraping dynamic, JavaScript-heavy websites.
    * It contains the logic to parse the messy HTML content and extract the desired data points (e.g., stock prices, news headlines, financial figures).
    * Crucially, after extracting the data, it connects directly to the MongoDB container over the internal Docker network and writes the data into the database.

#### **e. Container 5: The MongoDB Database (The Archive)**
* **Technology:** MongoDB Community Edition
* **Role:** The primary data store for the application.
* **Technical Details:**
    * It is a **NoSQL, document-oriented database**. This means it stores data in flexible, JSON-like documents.
    * **Why it's perfect for this project:** Web-scraped data is often semi-structured or unpredictable. One financial website might provide a "revenue" figure, while another calls it "total sales." A relational database (like PostgreSQL) would require a rigid, predefined schema, making it difficult to store such varied data. MongoDB's flexible schema allows us to store the data as-is, without losing information.
    * It is only accessible from within the internal Docker network, primarily by the Data Miner (for writing) and the Django App (for reading). It is completely firewalled from the public internet.

---

### **4. End-to-End Workflow Example**

Let's trace a single user request from start to finish:

1.  **Input:** The user navigates to `http://localhost` in their browser. The request hits the **Apache** container. Apache serves the Django-rendered HTML page with an input form.
2.  **Submission:** The user types "Scrape news from `financenews.com`" and clicks "Submit." The browser sends a `POST` request to Apache.
3.  **Proxy:** Apache forwards this request to the **Django** container.
4.  **Delegation:** The Django view receives the form data. It packages it into a JSON object and makes an internal `POST` request to `http://smart-router:5001/api/v1/submit-task`.
5.  **Routing:** The **Smart Router** receives the request. It identifies the task as a "scrape" job and makes a subsequent internal `POST` request to `http://data-miner:5002/api/v1/execute-scrape`, passing along the target URL.
6.  **Execution:** The **Data Miner** receives the instruction. It initiates a connection to `financenews.com`, downloads the HTML, parses it to extract all news headlines, and formats them into a list of JSON objects.
7.  **Storage:** The Data Miner connects to the **MongoDB** container (`mongodb://mongodb:27017`) and inserts the list of news headlines into a collection named `scraped_articles`.
8.  **Display:** The user navigates to the "Results" page. This sends a new request through Apache to Django. The corresponding Django view executes a query against the MongoDB database, fetches the latest articles from the `scraped_articles` collection, and renders them in an HTML template that is sent back to the user's browser.

This entire workflow demonstrates a clean, decoupled, and robust process for handling tasks without blocking the user interface.


what would your approach be?
