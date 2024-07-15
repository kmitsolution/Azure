# User Types and Azure Active Directory Integration

**User Types:**

1. **Azure AD Users**: Users managed within your organization's Azure Active Directory. They typically have organizational accounts and are integrated with Azure DevOps for seamless access management.

2. **External Users**: Users who are outside your organization's Azure AD or external to any connected directories. Their access may be limited when Azure DevOps is connected to a specific directory.

**Azure DevOps User Types:**

- **Basic User**: Ideal for users who need access to work items, Agile tools, and feedback requests. They can create and modify work items and view the dashboard.

- **Stakeholder**: Intended for users who need to track and view progress, provide feedback, and participate in discussions. Stakeholders have read-only access to most features.

- **Visual Studio Subscriber**: Users with active Visual Studio subscriptions gain access to comprehensive features in Azure DevOps, including full project access and advanced collaboration tools.

**Table: Differences Between User Types**

| Feature / Capability       | Basic User                            | Stakeholder                           | Visual Studio Subscriber              |
|----------------------------|---------------------------------------|---------------------------------------|----------------------------------------|
| View Boards and Backlogs   | Yes                                   | Yes                                   | Yes                                    |
| Create and Modify Work Items | Yes                                 | No (Read-only)                        | Yes                                    |
| Participate in Discussions | Yes                                   | Yes                                   | Yes                                    |
| View Repositories          | Yes                                   | Yes                                   | Yes                                    |
| View Build and Release      | Yes                                   | No                                    | Yes                                    |
| Access Project Dashboards  | Yes                                   | Yes                                   | Yes                                    |
| Access Analytics          | Yes                                   | No                                    | Yes                                    |
| Access to Test Plans       | Yes                                   | No                                    | Yes                                    |
| Access to Wiki             | Yes                                   | No                                    | Yes                                    |

**Considerations When Connecting Azure Active Directory:**

- **Loss of Access for External Users**: When Azure DevOps is connected to a specific directory like Azure AD, external users not included in that directory may lose access to Azure DevOps resources unless explicitly added as guests or external users within Azure AD.

---

This documentation structure provides clarity on user types, their capabilities within Azure DevOps, and considerations when integrating with Azure Active Directory. It also outlines the impact on external users' access when Azure DevOps is connected to a specific directory. Adjust the specifics based on your organization's Azure DevOps configuration and user management requirements.
