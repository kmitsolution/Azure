Certainly! Here's the full expanded assignment, incorporating all the steps and questions from setting up Azure DevOps to tracking tasks, bugs, and creating queries, as well as managing sprints, tasks, and work item states. Follow the steps sequentially for a comprehensive learning experience in Azure DevOps Boards.

---

### **Assignment: Azure DevOps Boards for Employee Management Project**

#### **Case Study: Employee Management System (EMS)**
The **Employee Management System (EMS)** is being developed for a company that requires managing employee data, roles, tasks, attendance, and performance. The system is developed using the **Agile methodology** and **Azure DevOps Boards** for managing work items, sprints, and progress.

---

### **Assignment Steps and Questions**

---

#### **Part 1: Set Up and Configuration**

1. **Create Users in Active Directory**
   - **Task**: Create two new users in Azure Active Directory:
     - **emp1**: Employee 1 (Developer)
     - **emp2**: Employee 2 (Product Manager)
   - **Steps**:
     - Navigate to Azure Active Directory.
     - Under "Users", select "New user" and enter details for **emp1** and **emp2**.
     - Assign roles as needed (emp1 as Contributor, emp2 as Product Owner).
   - **Explain** how to assign appropriate roles and permissions to these users within **Azure DevOps**.

2. **Set up a Public Project in Azure DevOps**
   - **Task**: Create a **public project** named **"EmployeeManagement"** in **Azure DevOps** and configure it to use the **Agile Process** template.
   - **Steps**:
     - Go to Azure DevOps and create a new project.
     - Choose the **Agile** template.
     - Add **emp1** and **emp2** to the project and assign them appropriate roles (Contributor for emp1, Product Owner for emp2).
   - **Explain** the differences between Agile, Scrum, and CMMI templates.

---

#### **Part 2: Create and Manage Work Items**

3. **Create Epics**
   - **Task**: Create at least 3 **Epics** related to the EMS project.
     - Example Epics:
       - Employee Onboarding
       - Employee Attendance Management
       - Employee Performance Evaluation
   - **Steps**:
     - Navigate to Boards > Backlog and create Epics.
   - **Provide** the title and description for each Epic.

4. **Create User Stories**
   - **Task**: For each Epic, create at least 2 **User Stories**.
     - Example for **Employee Onboarding** Epic:
       - "As an HR admin, I want to add new employee details to the system."
       - "As a user, I want to upload employee documents to the system."
   - **Steps**:
     - Create User Stories under each Epic and add relevant details.
   - **Provide** the title and description for each User Story.

5. **Create Tasks**
   - **Task**: Break down each User Story into at least 2 **Tasks**.
     - Example:
       - User Story: "As an HR admin, I want to add new employee details."
       - Tasks: "Design database schema" and "Develop UI for employee data entry."
   - **Steps**:
     - Add Tasks under each User Story.
   - **Provide** the titles and descriptions for each Task.

6. **Create Bugs**
   - **Task**: Create at least 2 **Bugs** related to the work items.
     - Example:
       - Bug: "Employee details form crashes when submitting incomplete data."
       - Bug: "Performance report shows incorrect attendance values."
   - **Steps**:
     - Create Bugs under the Bugs section in Azure DevOps.
   - **Provide** the titles and descriptions for each Bug.

---

#### **Part 3: Sprint Planning and Tracking**

7. **Create a Sprint**
   - **Task**: Create the first **Sprint** (e.g., Sprint 1) for the EMS project.
   - **Steps**:
     - Navigate to **Boards > Sprints**.
     - Create a new Sprint with defined start and end dates.
     - Assign at least 3 User Stories to Sprint 1, with appropriate team members.
   - **Explain** how to assign work items to a sprint and assign team members.

8. **Manage Backlog and Sprint Scope**
   - **Task**: Add at least 3 User Stories, 2 Epics, and 2 Tasks to the product backlog.
   - **Steps**:
     - Prioritize the backlog items and assign them to Sprint 1.
   - **Explain** how you would adjust sprint scope if work items are not completed within the sprint timeframe.

9. **Track Sprint Progress with Work Items**
   - **Task**: Update the state of work items in Sprint 1 (e.g., from New to In Progress, then Done).
   - **Steps**:
     - Change states for tasks assigned to emp1 and emp2.
     - Add comments, tags, or attachments if necessary.
   - **Explain** how you would monitor progress using work item states (New, In Progress, Done).

---

#### **Part 4: Queries and Filtering**

10. **Create a Query to Track Open Bugs**
    - **Task**: Create a query to display all **active Bugs** that have not been resolved yet.
    - **Steps**:
      - Go to **Boards > Queries** and create a new query.
      - Filter by **State = Active** and **Work Item Type = Bug**.
    - **Explain** how this query can help identify issues that need to be fixed.

11. **Create a Query to Track Open Work Items**
    - **Task**: Create a query to display all **open** work items (Tasks and User Stories) that are in New, Active, or In Progress states.
    - **Steps**:
      - Filter by **State** (New, Active, In Progress) and **Work Item Type** (Task, User Story).
    - **Explain** how tracking open work items helps to understand the remaining work in a sprint.

---

#### **Part 5: Task Assignment and Management**

12. **Change Task State from New to Active**
    - **Task**: Select a **Task** in the **New** state and change its state to **Active**.
    - **Steps**:
      - Navigate to the **Boards > Tasks** section.
      - Select a task and update the **State** to Active.
    - **Explain** why state transitions like **New** to **Active** are important for tracking progress.

13. **Create a Bug Related to a Task**
    - **Task**: Create a **Bug** related to a task that is in **Active** state.
    - **Example Bug**: "Bug: Task UI for employee details form is not responsive."
    - **Steps**:
      - Create a Bug and link it to the task.
    - **Explain** how linking Bugs to tasks helps to keep track of issues that arise during development.

14. **Assign Tasks to Different Developers**
    - **Task**: Assign at least **3 Tasks** to different developers (e.g., emp1 and emp2).
    - **Steps**:
      - Select tasks and assign them to **emp1** and **emp2**.
    - **Explain** how distributing tasks between team members helps manage sprint workload.

15. **Create a Query to Find Active/Closed Tasks for Each User**
    - **Task**: Create a query to find **Active** tasks assigned to **emp1** and **emp2**.
    - **Steps**:
      - Filter by **Assigned To** (emp1 or emp2) and **State** (Active).
    - **Create another query** to find **Closed** tasks for each user.
      - Filter by **State = Done** and **Assigned To** (emp1 or emp2).
    - **Explain** how these queries help track the individual progress of each team member.

---

#### **Part 6: Bug and Work Item Management**

16. **Move a Bug from Active to Resolved State**
    - **Task**: Select a **Bug** that is in **Active** state and move it to **Resolved** after it has been fixed.
    - **Steps**:
      - Update the Bug's **State** from **Active** to **Resolved**.
    - **Explain** the importance of tracking the lifecycle of a bug from Active to Resolved.

17. **Move a Task from Active to Done State**
    - **Task**: Select a **Task** (e.g., "Develop UI for adding employee details") and change its state to **Done** once completed.
    - **Steps**:
      - Update the **State** of the task to **Done**.
    - **Explain** the significance of completing and marking tasks as Done in Agile workflows.

---

#### **Part 7: Query Enhancements & Reporting**

18. **Create a Query to Find Overdue Work Items**
    - **Task**: Create a query to find **work items** that have passed their due dates and are still in an open state (New, Active, or In Progress).
    - **Steps**:
      - Filter by **Due Date** and **State** (New, Active, In Progress).
    - **Explain** how overdue work items impact sprint delivery and why it's important to track them.

19. **Create a Query for Work Items by Priority**
    - **Task**: Create a query to find **work items** ordered by **Priority** (e.g., High, Medium, Low).
    - **Steps**:
      - Filter by **Priority**

 and sort by **Work Item Type**.
    - **Explain** how prioritizing work items ensures the most important tasks are completed first.

---

#### **Part 8: Custom Dashboard and Reporting**

20. **Create a Custom Dashboard for Sprint Progress**
    - **Task**: Create a **Custom Dashboard** that includes:
      - **Burndown Chart** (work completed over time)
      - **Work Items by State** (pie chart showing New, Active, Done)
      - **Active Bugs** (list or graph showing bugs in Sprint 1)
      - **Work Items Assigned to emp1 and emp2** (pie chart showing task distribution)
    - **Steps**:
      - Navigate to **Dashboards > New Dashboard** and add visual widgets.
    - **Explain** how this dashboard helps monitor sprint health and track work item progress.

---

### **Bonus (Optional)**

21. **Linking Work Items (Tasks, Bugs, User Stories)**
    - **Task**: Link a **Bug** to a **Task** and a **Task** to a **User Story**.
    - **Steps**:
      - Link the **Bug** to the related **Task** (e.g., "Bug: Form crash" linked to "Develop UI for employee details").
      - Link the **Task** to the corresponding **User Story**.
    - **Explain** how linking work items maintains traceability and context.

22. **Track Multiple Sprints**
    - **Task**: If Sprint 1 is completed, create **Sprint 2** and assign new work items.
    - **Steps**:
      - Create **Sprint 2** and assign new User Stories and Tasks.
    - **Create a query** to track work items for **Sprint 2**.
    - **Explain** how Azure DevOps helps manage and report progress across multiple sprints.

---

### **Deliverables:**
1. **Screenshots** of work items (Tasks, Bugs, User Stories) with their state transitions.
2. **Queries** created for tracking open tasks, active bugs, and work items.
3. **Custom Dashboard** or visual reports showing sprint metrics.
4. **Explanations** of state changes, bug tracking, and task assignments.
5. **Query Configurations** and explanations for filtering work items based on priority, overdue, and state.

---

### **Notes:**
- This comprehensive assignment will give you practical experience in **task management**, **bug tracking**, **sprint planning**, and **reporting** in Azure DevOps.
- You'll learn how to manage work item states, assign tasks to team members, and track progress using queries and dashboards.
- The goal is to equip you with the skills to manage an Agile project effectively in Azure DevOps.

