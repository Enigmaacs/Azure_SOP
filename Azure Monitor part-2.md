# Azure Monitor Alerts and Action Groups - SOP

## Purpose
This Standard Operating Procedure (SOP) outlines the steps to configure alerts using Azure Monitor. It includes the setup of Action Groups with notifications and the creation of alert rules for Azure resources.

---

## Step 1: Create Action Group

1. **Navigate to Azure Monitor**
   - Go to the Azure portal.
   - Search for and select **Monitor**.

2. **Access Alerts Section**
   - In the Monitor pane, click **Alerts** from the left-side menu.
   - Select **Manage actions** > click **+ Create action group**.

3. **Basic Details**
   - **Subscription**: Choose the appropriate subscription.
   - **Resource Group**: Create a new one or use an existing one.
   - **Action Group Name**: Provide a unique name (e.g., `AlertAdminGroup`).
   - **Display Name**: A user-friendly name (e.g., `Admin Alerts`).

4. **Notifications**
   - Click **+ Add notification**.
   - **Notification Type**:
     - **Email Azure Resource Manager Role**: Notifies users with specific ARM roles at subscription, resource group, or resource level.
     - **Email/SMS/Push/Voice**: Manually add recipients.
   - For manual notifications:
     - Choose **Email**.
     - Enter the recipient's email address.
     - Provide a name (e.g., `EmailAlertAdmin`).
     - Enable **Common Alert Schema** → **Yes**.
     - Click **OK**.

5. **Actions (Optional)**
   - You can add automated responses like:
     - Logic App
     - Azure Function
     - Webhook
   - Click **+ Add action** if required.

6. **Tags (Optional)**
   - Add tags to manage and organize Action Groups.

7. **Review + Create**
   - Click **Review + Create** → **Create** to finish setup.

---

## Step 2: Create an Alert Rule

1. **Select Target Resource**
   - Go to the resource you want to monitor (e.g., VM, Log Analytics).
   - In the left menu, click **Alerts** > **+ Create** > **Alert rule**.

2. **Scope**
   - Confirm or select the resource.

3. **Condition**
   - Click **Add condition**.
   - Choose a signal (e.g., CPU > 80%, log result).
   - Set the threshold and evaluation frequency.

4. **Actions**
   - Click **Add action group**.
   - Select the Action Group created earlier (e.g., `AlertAdminGroup`).

5. **Alert Rule Details**
   - Provide a name (e.g., `HighCPUAlert`).
   - Add a description (optional).
   - Choose **Severity** (0 = Critical, 4 = Informational).
   - Enable the rule on creation if needed.

6. **Review + Create**
   - Review all settings.
   - Click **Create alert rule**.

---

## Step 3: Confirm Email Subscription

- If the email address is used for the first time in an Action Group:
  - Azure sends a confirmation email.
  - The recipient must **click the confirmation link**.
  - Alerts will only be received after confirmation.

---

## Notes

- Action Groups are reusable across multiple alerts.
- Always enable the **Common Alert Schema** for standardized alert formatting.
- Push notifications require the **Azure mobile app** to be installed and signed in.

---

_This SOP helps ensure reliable and standardized alert configurations across Azure environments._

