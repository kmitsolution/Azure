# Scenario

Suppose your company has offices in:

* 🇮🇳 India
* 🇺🇸 United States

There are two Help Desk teams:

| Help Desk       | Should Manage               |
| --------------- | --------------------------- |
| India Help Desk | Only India users and groups |
| US Help Desk    | Only US users and groups    |

The India Help Desk **must not** be able to:

* Reset passwords of US users
* Create US users
* Delete US users
* Manage US groups

Similarly, the US Help Desk should only manage US identities.

This is called **Delegated Administration**.

---

# Solution Architecture

```text
Microsoft Entra ID

│
├── Administrative Unit
│      India
│
│      ├── Users
│      ├── Groups
│      └── Help Desk Admin
│
└── Administrative Unit
       USA

       ├── Users
       ├── Groups
       └── Help Desk Admin
```

Notice:

**Administrative Unit does NOT contain Azure Resources.**

It contains only identity objects like:

* Users
* Groups
* Devices (optional)

---

# Step 1: Create Users

Go to

```
Microsoft Entra ID

↓

Users
```

Create users.

Example:

| Name  | Country |
| ----- | ------- |
| Raman | India   |
| Rahul | India   |
| Amit  | India   |
| John  | USA     |
| David | USA     |
| James | USA     |

---

# Step 2: Create Help Desk Users

Create two administrators.

```
IndiaHelpDesk

USHelpDesk
```

These are normal users for now.

---

# Step 3: Create Groups

Create groups.

```
India-Employees

US-Employees
```

Add members.

India Group

```
Raman

Rahul

Amit
```

US Group

```
John

David

James
```

---

# Step 4: Create Administrative Unit (India)

Go to

```
Microsoft Entra ID

↓

Administrative Units

↓

New Administrative Unit
```

Enter

```
Name

India AU

Description

India Users and Groups
```

Click

```
Create
```

---

# Step 5: Create USA Administrative Unit

Again

```
Administrative Units

↓

New
```

Create

```
USA AU
```

---

# Step 6: Add Users to India AU

Open

```
India AU

↓

Users

↓

Add
```

Select

```
Raman

Rahul

Amit
```

Now

```
India AU

↓

Users

↓

Raman

Rahul

Amit
```

---

# Step 7: Add Groups

Open

```
India AU

↓

Groups

↓

Add
```

Select

```
India-Employees
```

Now the AU contains:

```
India AU

Users

Groups
```

Do the same for USA AU.

---

# Step 8: Assign Administrator

This is the important step.

Open

```
India AU

↓

Roles and administrators
```

Click

```
Add Assignment
```

Choose role.

Example

```
Helpdesk Administrator
```

Select

```
IndiaHelpDesk
```

Click

```
Assign
```

Now

```
IndiaHelpDesk

↓

Helpdesk Administrator

↓

Scope

↓

India AU
```

Notice:

The role is **not Global**.

It is limited to the India Administrative Unit.

---

# Step 9: Configure USA

Do the same.

```
USA AU

↓

Roles

↓

Helpdesk Administrator

↓

USHelpDesk
```

---

# Final Architecture

```
Microsoft Entra ID

│

├── India AU

│      │

│      ├── Raman

│      ├── Rahul

│      ├── India Employees Group

│      │

│      └── IndiaHelpDesk

│            Helpdesk Administrator

│

└── USA AU

       │

       ├── John

       ├── David

       ├── USA Employees Group

       │

       └── USHelpDesk

             Helpdesk Administrator
```

---

# What Can India Help Desk Do?

Suppose IndiaHelpDesk logs in.

He can:

✅ Reset Raman password

✅ Reset Rahul password

✅ Update India users

✅ Manage India groups

---

He cannot:

❌ Reset John's password

❌ Delete David

❌ Modify USA groups

---

# Test the Scenario

Login as

```
IndiaHelpDesk
```

Open

```
Users
```

You'll only be able to administer the users that are in the **India AU** according to the delegated role.

Try

```
John
```

You'll receive insufficient permissions because John belongs to the **USA AU**.

---

# Common Roles You Can Scope to an Administrative Unit

You can assign roles such as:

* Helpdesk Administrator
* User Administrator
* Groups Administrator
* Password Administrator
* Authentication Administrator
* License Administrator (where supported)

These roles apply **only within the Administrative Unit**.

---

# Difference Between Administrative Unit and Group

| Group                                 | Administrative Unit                                      |
| ------------------------------------- | -------------------------------------------------------- |
| Used to assign permissions to members | Used to delegate administration                          |
| Members get access to resources       | Administrators get scoped management rights              |
| Used with Azure RBAC and licensing    | Used with Entra administrative roles                     |
| Contains users, devices, groups       | Contains users, groups, devices for administrative scope |

---

# Interview Question ⭐

**Interviewer:**

> Why not simply create an India Group and assign the Helpdesk Administrator role to it?

**Answer:**

Because a **Group** is used to organize members or assign access to resources. If you assign the **Helpdesk Administrator** role at the tenant level to that group, the administrators would have tenant-wide help desk permissions.

An **Administrative Unit** lets you **scope** the administrative role so that the administrator can manage **only the users, groups, and devices within that Administrative Unit**, not the entire tenant.

---

## Enterprise Example

Companies like Microsoft, TCS, Infosys, and Accenture commonly create Administrative Units based on:

* India
* USA
* Europe
* Australia
* Japan

Each regional IT team receives delegated roles only for its own Administrative Unit, allowing regional administration without granting unnecessary tenant-wide privileges.

This approach follows the **principle of least privilege**, reducing security risk while enabling local support teams to manage their own users and groups.
