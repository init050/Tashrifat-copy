# Architecture Overview

This document provides a high-level overview of the technical architecture for the Tashrifat Saadat project.

## 1. High-Level Design

The application is a monolithic web application built with the Django framework. It follows a traditional server-side rendering model.

### ASCII Diagram

```
+-----------------+      +----------------------+      +----------------+
|   User/Browser  | <--> |      Web Server      | <--> |   Django App   |
+-----------------+      | (e.g., Gunicorn)     |      | (Tashrifat)    |
                         +----------------------+      +-------+--------+
                                                               |
                                                               v
+--------------------------------------------------------------+
|                                                              |
|   +-----------------+      +----------------------+          |
|   |  Static Files   |      |      Media Files     |          |
|   | (CSS, JS, Img)  |      |   (User Uploads)     |          |
|   +-----------------+      +----------------------+          |
|                                                              |
|   +--------------------------------------------------------+ |
|   |                        Database                        | |
|   |                      (SQLite/PostgreSQL)                 | |
|   +--------------------------------------------------------+ |
|                                                              |
+-------------------------File System--------------------------+

```

## 2. Database Schema

The database schema is defined using the Django ORM. The main models are:

- **`Service` (`services_module`):** The core model representing a service category. It includes fields for title, description, pricing, and guest capacity.
- **`Menu` & `MenuItem` (`services_module`):** These models define the menus and items available for each service.
- **`Gallery` (`services_module`):** Stores images associated with services, categorized for different uses (e.g., main gallery, slider).
- **`Contact` (`contact_module`):** A simple model to store messages submitted through the contact form.
- **`SiteSetting` (`site_module`):** A singleton-like model for storing global site settings such as the site name, logo, and contact information.
- **`User` (`django.contrib.auth`):** The standard Django User model is used for admin authentication.

## 3. Authentication Flow

Authentication is handled by Django's built-in authentication system.

- **Admin Users:** Authenticate through the standard Django admin login page (`/admin`). Once authenticated, they are granted access to the admin panel to manage site content. Session cookies are used to maintain the authenticated state.
- **Public Users:** There is no authentication flow for public-facing users (customers). They can browse the site and services without needing to log in.

## 4. Roles

The system has two primary roles:

- **Administrator:** A superuser or staff user with access to the Django admin panel. They have full CRUD (Create, Read, Update, Delete) permissions on all database models. Their primary responsibility is to manage the website's content, including services, menus, gallery images, and site settings.
- **Anonymous User:** A typical visitor to the website. They have read-only access to the public pages of the site, such as the home page, service listings, and contact page. They can submit messages through the contact form.

## 5. Data Flow

The data flow is characteristic of a server-side rendered application:

1.  A user's browser sends an HTTP request to a specific URL (e.g., `/services/catering/`).
2.  The web server (like Gunicorn or Nginx) forwards the request to the Django application.
3.  Django's URL dispatcher matches the URL to a specific view function.
4.  The view function interacts with the Django ORM to fetch the necessary data from the database (e.g., retrieves the "catering" service details).
5.  The view function passes the data to a Django template.
6.  The template engine renders the data into an HTML document.
7.  The Django application sends the generated HTML back to the user's browser as an HTTP response.
8.  The browser parses the HTML and renders the final page, fetching any linked static assets (CSS, JavaScript, images) in subsequent requests.

## 6. Architectural Trade-offs

The choice of a monolithic Django architecture comes with several trade-offs:

### Advantages

- **Simplicity & Rapid Development:** Django's "batteries-included" nature and the monolithic structure make it easy to develop, test, and deploy. Features can be added quickly without the complexity of coordinating multiple services.
- **Consistency:** The entire codebase is in one repository, using a single language (Python) and framework (Django). This ensures a consistent development style and reduces the learning curve for new developers.
- **Performance:** For many use cases, a monolithic application can be more performant than a microservices architecture because it avoids the network latency and serialization overhead of inter-service communication.

### Disadvantages

- **Scalability:** While the application can be scaled vertically (by using a more powerful server), horizontal scaling (adding more servers) can be less efficient. The entire application must be scaled, even if only one component is a bottleneck.
- **Technology Lock-in:** The tightly coupled nature of the monolith makes it difficult to adopt new technologies or rewrite specific components in a different language or framework.
- **Deployment Complexity:** As the application grows, deployments can become slower and riskier. A bug in one module can potentially bring down the entire application.
