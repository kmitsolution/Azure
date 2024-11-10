To create and manage Azure resource groups, add users to them, and assign roles with specific permissions, you can use **Azure CLI** or the **Azure Portal**. I'll walk you through both methods for each task. 

### **Task Overview**:
1. Create **Dev Group** and **Test Group**.
2. Add the user `dev1` to the **Dev Group** with the **Contributor** role (allowing full access to create and delete virtual machines).
3. Add the user `test1` to the **Test Group** with **Reader** role (only read-only access).

---

### **Using Azure CLI:**

#### Step 1: **Log in to Azure CLI**
Make sure you're logged in to your Azure account using Azure CLI:
```bash
az login --use-device
```

#### Step 2: **Create Resource Groups (optional)**
You need resource groups to add users and roles within. If you don't have resource groups already, you can create them using the following commands:
```bash
az group create --name DevGroup --location EastUS
az group create --name TestGroup --location EastUS
```

#### Step 3: **Create Azure Active Directory Groups**

To create AAD groups for Dev and Test users:
```bash
az ad group create --display-name "Dev Group" --mail-nickname "devgroup"
az ad group create --display-name "Test Group" --mail-nickname "testgroup"
```

#### Step 4: **Add Users to Groups**

If the users `dev1` and `test1` already exist in your directory, you can add them to the groups as follows:

1. **Add `dev1` to Dev Group:**
   ```bash
   az ad group member add --group "Dev Group" --member-id dev1
   ```

2. **Add `test1` to Test Group:**
   ```bash
   az ad group member add --group "Test Group" --member-id test1
   ```

> **Note:** Replace `dev1` and `test1` with the **Object IDs** of the users if needed. You can find the Object ID for a user using:
```bash
az ad user show --id dev1 --query objectId
```

#### Step 5: **Assign Roles to Users in Groups**

Next, you assign specific roles to the users in their respective groups.

1. **Assign Contributor Role to `dev1` in the Dev Group**:
   The **Contributor** role allows users to create and manage virtual machines. You will assign the role at the subscription or resource group level.

   ```bash
   az role assignment create --assignee dev1 --role "Contributor" --scope /subscriptions/{subscriptionId}/resourceGroups/DevGroup
   ```

   Replace `{subscriptionId}` with your actual Azure subscription ID.

2. **Assign Reader Role to `test1` in the Test Group**:
   The **Reader** role only provides read-only access. Here's how you can assign the role:
   
   ```bash
   az role assignment create --assignee test1 --role "Reader" --scope /subscriptions/{subscriptionId}/resourceGroups/TestGroup
   ```

#### Step 6: **Verify Role Assignments**
You can verify that the roles have been correctly assigned by running the following command:

```bash
az role assignment list --assignee dev1 --all
az role assignment list --assignee test1 --all
```

These commands will show all roles assigned to the respective users.

---

### **Using Azure Portal:**

#### Step 1: **Create Resource Groups**
1. **Go to Azure Portal**: [portal.azure.com](https://portal.azure.com).
2. **Create Resource Groups**: 
   - In the left-hand sidebar, search for **Resource Groups** and click on it.
   - Click on **+ Add**.
   - Enter the name of the resource group as `DevGroup` and select the region (e.g., **East US**).
   - Click **Review + Create** and then **Create**.
   - Repeat for `TestGroup`.

#### Step 2: **Create AAD Groups (Dev and Test Groups)**

1. **Go to Azure Active Directory**:
   - In the left-hand sidebar, search for **Azure Active Directory**.
   
2. **Create Dev Group**:
   - In the Azure AD menu, select **Groups** > **+ New Group**.
   - Choose **Security** as the group type.
   - Name the group "Dev Group", and add the `dev1` user as a member.
   - Click **Create**.

3. **Create Test Group**:
   - Repeat the same process to create the "Test Group" and add `test1` as a member.

#### Step 3: **Assign Roles to Users in Groups**

1. **Assign Contributor Role to `dev1` in Dev Group**:
   - Go to **Resource Groups** in the Azure portal.
   - Click on `DevGroup`.
   - In the left-hand menu, click on **Access Control (IAM)**.
   - Click **+ Add** > **Add role assignment**.
   - In the **Role** field, select **Contributor**.
   - In the **Assign Access to** field, select **User, group, or service principal**.
   - Type "Dev Group" (or the name of the group), select it, and click **Save**.

2. **Assign Reader Role to `test1` in Test Group**:
   - Go to **Resource Groups** in the Azure portal.
   - Click on `TestGroup`.
   - In the left-hand menu, click on **Access Control (IAM)**.
   - Click **+ Add** > **Add role assignment**.
   - In the **Role** field, select **Reader**.
   - In the **Assign Access to** field, select **User, group, or service principal**.
   - Type "Test Group" (or the name of the group), select it, and click **Save**.

#### Step 4: **Verify the Roles**

- You can verify the roles in the **Access Control (IAM)** section of each resource group to ensure the correct roles are assigned.

---

### **Summary of Tasks**

1. **Create groups** in Azure Active Directory: Dev Group and Test Group.
2. **Add users** (`dev1` and `test1`) to the respective groups.
3. **Assign roles**:
   - **Contributor** role to `dev1` in the Dev Group, allowing full access to create and delete VMs.
   - **Reader** role to `test1` in the Test Group, providing read-only access.
4. **Verify** that roles are correctly assigned.

---

Let me know if you need further clarification or help with any of the steps!
