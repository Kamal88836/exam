# Red Hat EX280 Exam Questions

**Exam Details:**
- Total marks: 300
- Duration: 3 Hours
- Pass Marks: 210

---

## 1. Configure the HTPasswd identity provider

Configure your OpenShift cluster to use an HTPasswd identity provider with the following requirements.

- The name of the identity provider is **ex280-htpasswd**.
- The name of the secret is **ex280-idp-secret**.
- The user account **armstrong** is present and can log in with password **indiaco**.
- The user account **collins** is present and can log in with password **verastar**.
- The user account **aldrin** is present and can log in with password **moonhar**.
- The user account **jobs** is present and can log in with password **eastiver**.
- The user account **wozniak** is present and can log in with password **glimpse**.

---

## 2. Configure cluster permission

Configure your OpenShift cluster to meet the following requirements:

- The user account **jobs** can perform cluster administration tasks.
- The user account **wozniak** can create projects.
- The user account **wozniak** cannot perform cluster administration tasks.
- The user account **armstrong** cannot create projects.
- The user account **kubeadmin** is not present.

---

## 3. Configure project permissions

Configure your OpenShift cluster to meet the following requirements:

- Create the projects if not exists:
  - apollo
  - manhattan
  - gemini
  - bluebook
  - titan

- The user account **armstrong** is an administrator for project **apollo** and project **gemini**.
- The user account **wozniak** can view project **titan** but not administer or delete it.

---

## 4. Configure groups and permission

Configure your OpenShift cluster to meet the following requirements:

- The user account **armstrong** is a member of the **commander** group
- The user account **collins** is a member of the **pilot** group
- The user account **aldrin** is a member of the **pilot** group
- Members of the **commander** group have **edit** permission in the **apollo** project
- Members of the **pilot** group have **view** permission in the **apollo** project

---

## 5. Configure quotas

Configure your OpenShift cluster to use quotas in the **manhattan** project with the following requirements:

- The name of the quota is **ex280-quota**
- The amount of memory consumed across all containers may not exceed **1Gi**
- The total amount of CPU consumed across all containers may not exceed **1 full core**
- The maximum number of replication controllers does not exceed **3**
- The maximum number of pods does not exceed **3**
- The maximum number of services does not exceed **2**

---

## 6. Configure the secure route

Configure and deploy a secure route. Deploy an application called **oxcart** securely in the project called **area51**.

- The application has self-signed certificate available at:
  `/C=US/ST=NC/L=Raleigh/O=RedHat/OU=RHT/CN=oxcart.apps.ocp4.example.com`
- Use newcert command to generate required tls objects, pass the subject as argument value.
- The application should be reachable at **https://oxcart.apps.ocp4.example.com**
- Application produces a valid Output.

---

## 7. Deploy the helm chart application

Deploy the chart named **etherpad** in the project **aeti-service** from the repository:
`http://helm.ocp4.example.com/charts`

Note: You may use the telnet or nc commands to validate the deployment.

---

## 8. Configure a service account

Configure a service account in the **apples** project to meet the following requirements:

- The name of the service account is **ex280sa**
- The service account allows pods to be run as **any available user id**

---

## 9. Deploy the application

Deploy the application called **oranges** in the **apples** project so that the following conditions are true:

- The application uses the **ex280sa** service account
- No configuration components have been added or removed
- The application produces output.

---

## 10. Configure a Secret

Configure a secret in the **math** project and the name of the secret should be **magic**.

- The secret should have following key value pairs:
  - **ACCESS_STRING: ASDA142hfh-gfrhhueo-erfdk345v**

---

## 11. Configure an application to use a secret

Configure the application called **red** in the **math** project with the following requirements:

- The application uses the secret previously created called **magic**
- The secret defines an environment variable with name **ACCESS_STRING**
- The application output no longer displays "Sorry, application is not configured correctly"

---

## 12. Configure a network policy

Configure a network policy using the **database** and **checker** projects with the following requirements:

- The database project has a network policy with the name **db-allow-mysql-conn** based on pod selector label **network.openshift.io/policy-group**
- Connections to the database project are restricted to deployments from the checker project
- The network policy is filtered by Project selector using the **team=devsecops** label
- Pod selector using the **deployment=web-mysql** label.
- The application can establish a connection to port **3306/TCP**
- You can check your work by examining the logs in the checker project.

---

## 13. Deploy an application

Deploy the application called **atlas** in the **mercury** project so that the following conditions are true:

- No configuration components have been added or removed.
- The application produces output.

---

## 14. Install an operator

Install the **file-integrity** operator with the following requirements:

- The operator is installed in the **openshift-file-integrity** project
- The approval strategy is **Automatic**
- Cluster monitoring is enabled for the **openshift-file-integrity** project

---

## 15. Configure Template

Configure your OpenShift cluster so that new projects are created with limits using the following requirements:

- The name of the limit range is **PROJECT_NAME-limits**, where **PROJECT_NAME** is the name of the project created using `oc new-project`.
- The amount of memory consumed by a single container is:
  - Minimum: **128Mi**
  - Maximum: **1Gi**
  - Default: **512Mi**
  - Default request: **256Mi**

---

## 16. Create a cron job

Create a cron job with the following requirements:

- The cron job name is **ipsum**
- The cron job uses the image **quay.io/redhattraining/scaling**
- The cron job runs **every minute**
- The cron job successful job history limit is **10**
- The cron job runs in a project called **lorem**

---

## 17. Scale the Application manually

Scale an application called **hydra** in the project called **lerna**. The **hydra** application should be scaled to **five times**.

---

**Best of Luck!**
