# Session-Puzzling

What is Zabbix?
In short, Zabbix is an enterprise-class, open-source monitoring solution. It is used to track the health, performance, and availability of network servers, devices, virtual machines, and cloud services.

In the context of your pentesting, you are interacting with the Zabbix Frontend (the PHP-based web UI). This interface is what manages the session variables you are trying to "puzzle."

## Session Puzzling in Zabbix
Session Puzzling (or Session Variable Overloading) occurs when an application uses a single session variable for multiple purposes. For example, if Zabbix stored a variable like $_SESSION['target_user_id'] and failed to clear it between different modules, an action in one tab could accidentally apply to a user selected in another tab.

## Clean End-to-End Test Plan
To test this in the latest Zabbix (v6.x, v7.0, or the latest v8.0), focus on modules that involve multi-step wizards or mass updates, as these are the most likely candidates for session variable reuse.

### Phase 1: Preparation
**Target:** A Zabbix instance where you have Admin or Super Admin rights.

**Tools:** Two browser tabs (same session/browser profile) and Burp Suite.

**Setup:** Create two dummy users: Victim_A and Victim_B.

### Phase 2: The Attack Execution
**Tab A (The Start):** Go to Administration -> Users. Click on Victim_A to edit their profile. Change a minor detail (like their alias) but do not click "Update" yet.

**Tab B (The Context Switch):** In the same browser, open a new tab. Go to Administration -> Users and click on Victim_B. Perform a different action, such as viewing their Media or Permissions tab.

**Goal:** You are trying to force the server to update the "current user being edited" variable in your session to Victim_B.

**Tab A (The Completion):** Switch back to Tab A (which still shows Victim_A's info). Click the Update button.

### Phase 3: Validation
**PASS:** If Victim_A is updated and Victim_B remains unchanged. This means Zabbix uses unique IDs (usually passed in the POST request as userids[]) rather than relying on a generic "active user" variable stored in the server-side session.

**FAIL:** If Victim_B’s profile was modified with the data you intended for Victim_A. This indicates the server "got puzzled" and applied the save action to the last user ID loaded into the session.

### Advanced Zabbix-Specific Test: The "Mass Update"
Zabbix uses a "Mass Update" feature for hosts and items. This is a prime spot for session puzzling:

**Tab A:** Select 5 hosts and click Mass Update. Reach the screen where you choose which properties to change.

**Tab B:** Select a different set of 5 hosts and click Mass Update.

**Tab A:** Complete the update.

Check: Did the update apply to the hosts in Tab A or Tab B? If it applied to Tab B's hosts while you were in Tab A, you've found a Session Puzzling vulnerability.

Pro Tip: In modern Zabbix versions, you will likely find that they pass the sid (session ID) and specific ids in every request body, which is a strong defense against this. If you see userids=12 in your Burp Suite Repeater tab, the app is likely safe.
