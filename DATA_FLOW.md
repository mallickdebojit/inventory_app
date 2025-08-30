# Data Flow Diagram: Inventory-Pro

This diagram shows the flow of information between the user, the frontend application, the backend services, the database, and the AI processing pipeline.

```mermaid
graph TD
    subgraph "User's Browser"
        A[User] --> B{React/Next.js Frontend};
    end

    subgraph "Backend Infrastructure (Server)"
        B -- "1. API Request (HTTPS)" --> C{Django / DRF};
        C -- "2. Authenticate & Authorize" --> D[Middleware];
        D -- "3. Process Logic" --> E{Django Views};
        E -- "4. Database Query (ORM)" --> F[PostgreSQL Database];
        F -- "Stores/Retrieves Core Data & Dynamic JSONB" --> E;

        subgraph "Asynchronous AI Processing"
            E -- "5a. Dispatch Heavy Task" --> G[Celery Task Queue (Redis)];
            H[Celery Worker] -- "6. Fetch Task" --> G;
            H -- "7. Run AI Model (PyTorch/etc.)" --> I[AI Models];
            H -- "8. Store Result" --> F;
        end

        subgraph "Real-time AI Inference"
            E -- "5b. Quick AI Query (async)" --> J[Real-time AI Service / API];
            J -- "Returns Inference" --> E;
        end

        E -- "9. Serialize Response" --> C;
        C -- "10. API Response (JSON)" --> B;
    end

    B -- "11. Render UI Update" --> A;

    %% Styling
    classDef frontend fill:#D6EAF8,stroke:#333,stroke-width:2px;
    classDef backend fill:#D5F5E3,stroke:#333,stroke-width:2px;
    classDef db fill:#FADBD8,stroke:#333,stroke-width:2px;
    classDef ai fill:#FEF9E7,stroke:#333,stroke-width:2px;

    class B,A frontend;
    class C,D,E backend;
    class F db;
    class G,H,I,J ai;
```

### Explanation of the Flow:

1.  **API Request:** The user performs an action in the React frontend (e.g., adds a new item). The frontend sends a structured API request to the Django backend.
2.  **Authentication:** Django's middleware checks if the user is logged in and has the correct permissions for the requested tenant.
3.  **Business Logic:** The request is handled by the appropriate Django View, which contains the business logic.
4.  **Database Interaction:** The view uses the Django ORM to query the PostgreSQL database. It reads/writes standard columns and the flexible `JSONB` field for custom attributes.
5.  **AI Task Handling:**
    *   **5a (Asynchronous):** For a heavy, long-running task like "generate a sales forecast," the view dispatches a job to a Celery queue and immediately returns a "task started" response to the user.
    *   **5b (Real-time):** For a quick task like a natural language search, the view might `await` a direct call to a lightweight AI model or an external API.
6.  **Celery Worker:** A separate Celery process picks up the task from the queue.
7.  **AI Model Execution:** The worker runs the necessary AI computations.
8.  **Store Result:** The result of the AI job is saved back to the database.
9.  **API Response:** The Django view formats the data into a JSON response.
10. **Data to Frontend:** The JSON response is sent back to the React frontend.
11. **UI Update:** The frontend uses the response data to update the UI, showing the user the result of their action.