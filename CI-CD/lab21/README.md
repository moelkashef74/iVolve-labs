# Lab 21: Role-based Authorization

## Lab Requirements

- Create `user1` and `user2`.
- Assign **admin** role for `user1` and **read-only** role for `user2`.

---

# Context

This lab configures fine-grained access control on the same Jenkins EC2 instance used in Labs 21/22 (provisioned via Terraform, configured via Ansible), using Jenkins's **Role-based Authorization Strategy** plugin instead of the default "Logged-in users can do anything" mode. Once configured:

- `user1` gets full administrative access (can configure Jenkins, manage credentials, create/edit/delete jobs, trigger builds, etc.)
- `user2` gets read-only access (can view jobs, builds, and console output, but cannot configure or trigger anything)

---

# Requirements from Previous Labs

- A running Jenkins instance on the EC2 host from Lab 22, accessible at `http://<jenkins-ec2-public-ip>:8080`.
- An account with **Administer** permissions (e.g. the initial `admin` user) to perform the setup below.

---

# Step 1: Install the Role-based Authorization Strategy Plugin

1. Go to **Manage Jenkins → Plugins → Available plugins**.
2. Search for `Role-based Authorization Strategy`.
3. Select it and click **Install without restart**.


---

# Step 2: Enable Role-Based Strategy as the Authorization Method

1. Go to **Manage Jenkins → Security → Configure Global Security**.
2. Under **Authorization**, select **Role-Based Strategy**.
3. Click **Save**.

---

# Step 3: Create user1 and user2

1. Go to **Manage Jenkins → Users → Create User**.
2. Fill in the form for the first user:
   - Username: `user1`
   - Password / Confirm password: `<choose a password>`
   - Full name: `User One`
   - Email: `user1@example.com`
3. Click **Create User**.
4. Repeat for the second user:
   - Username: `user2`
   - Password / Confirm password: `<choose a password>`
   - Full name: `User Two`
   - Email: `user2@example.com`


---

# Step 4: Create the admin and read-only Roles

1. Go to **Manage Jenkins → Manage and Assign Roles → Manage Roles**.
2. Under **Global roles**, add two roles:
   - `admin`
   - `read-only`
3. Configure the permission checkboxes for each role:

| Permission group | `admin` role                                             | `read-only` role     |
|-------------------|-----------------------------------------------------------|------------------------|
| Overall            | Read, **Administer**                                       | Read                    |
| Job                 | Build, Cancel, Configure, Create, Delete, Read, Workspace  | Read, Workspace         |
| Run                 | Delete, Replay, Update                                     | *(none)*                |
| View                | Create, Configure, Delete                                   | *(none)*                |
| SCM                 | Tag                                                          | *(none)*                |

4. Click **Save**.



---

# Step 5: Assign Roles to Users

1. Go to **Manage Jenkins → Manage and Assign Roles → Assign Roles**.
2. Under **Global roles**, click **Add user** and add `user1` and `user2`.
3. Check the box for `user1` under the **admin** column.
4. Check the box for `user2` under the **read-only** column.
5. Click **Save**.

---

# Step 6: Verify the Role Assignments

## 6.1 Log in as user1 (admin)

Log out of the admin account and sign in as `user1`.

```text
Username: user1
Password: <user1's password>
```

Confirm `user1` can see **Manage Jenkins** in the sidebar and can create/configure/delete jobs and trigger builds.


## 6.2 Log in as user2 (read-only)

Log out, then sign in as `user2`.

```text
Username: user2
Password: <user2's password>
```

Confirm `user2` **cannot** see "Manage Jenkins" or "New Item", but can still view job status, build history, and console logs.

