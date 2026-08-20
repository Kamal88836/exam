# Red Hat EX280 Exam - Complete Study Guide with Questions & Answers

**Exam Details:**
- Total marks: 300
- Duration: 3 Hours
- Pass Marks: 210

---

## **QUESTION 1: Configure the HTPasswd Identity Provider**

### Question:
Configure your OpenShift cluster to use an HTPasswd identity provider with the following requirements.

- The name of the identity provider is **ex280-htpasswd**.
- The name of the secret is **ex280-idp-secret**.
- The user account **armstrong** is present and can log in with password **indiaco**.
- The user account **collins** is present and can log in with password **verastar**.
- The user account **aldrin** is present and can log in with password **moonhar**.
- The user account **jobs** is present and can log in with password **eastiver**.
- The user account **wozniak** is present and can log in with password **glimpse**.

### Answer:

```bash
# Step 1: Create the HTPasswd file with users and passwords
htpasswd -c -B -b /tmp/htpasswd armstrong indiaco
htpasswd -B -b /tmp/htpasswd collins verastar
htpasswd -B -b /tmp/htpasswd aldrin moonhar
htpasswd -B -b /tmp/htpasswd jobs eastiver
htpasswd -B -b /tmp/htpasswd wozniak glimpse

# Step 2: Create the secret in openshift-config project
oc create secret generic ex280-idp-secret \
  --from-file=htpasswd=/tmp/htpasswd \
  -n openshift-config

# Step 3: Create/Update the OAuth CR
cat <<EOF | oc apply -f -
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: ex280-htpasswd
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: ex280-idp-secret
EOF

# Step 4: Verify the configuration
oc get oauth cluster -o yaml
oc login -u armstrong -p indiaco
oc whoami
```

**Key Points:**
- `-c` creates a new file
- `-B` uses bcrypt encryption
- `-b` takes password as argument (non-interactive)
- `mappingMethod: claim` ensures users are created with their HTPasswd username
- Secret must be in `openshift-config` namespace

---

## **QUESTION 2: Configure Cluster Permission**

### Question:
Configure your OpenShift cluster to meet the following requirements:

- The user account **jobs** can perform cluster administration tasks.
- The user account **wozniak** can create projects.
- The user account **wozniak** cannot perform cluster administration tasks.
- The user account **armstrong** cannot create projects.
- The user account **kubeadmin** is not present.

### Answer:

```bash
# Step 1: Add cluster-admin role to jobs
oc adm policy add-cluster-role-to-user cluster-admin jobs

# Step 2: Add self-provisioner role to wozniak (allows creating projects)
oc adm policy add-cluster-role-to-user self-provisioner wozniak

# Step 3: Ensure wozniak does NOT have cluster-admin
oc adm policy remove-cluster-role-from-user cluster-admin wozniak

# Step 4: Ensure armstrong cannot create projects (default - no action needed)
# Verify armstrong is NOT a self-provisioner
oc adm policy remove-cluster-role-from-user self-provisioner armstrong 2>/dev/null || true

# Step 5: Remove kubeadmin secret
oc delete secret kubeadmin -n kube-system

# Step 6: Verify permissions
oc get clusterrolebinding | grep jobs
oc get clusterrolebinding | grep wozniak
```

**Key Points:**
- `cluster-admin` = full cluster administration
- `self-provisioner` = can create projects
- Removing kubeadmin secret disables the emergency admin account
- Use `oc adm policy` for role management

---

## **QUESTION 3: Configure Project Permissions**

### Question:
Configure your OpenShift cluster to meet the following requirements:

- Create the projects if not exists:
  - apollo
  - manhattan
  - gemini
  - bluebook
  - titan

- The user account **armstrong** is an administrator for project **apollo** and project **gemini**.
- The user account **wozniak** can view project **titan** but not administer or delete it.

### Answer:

```bash
# Step 1: Create all required projects
oc new-project apollo --display-name="Apollo Project"
oc new-project manhattan --display-name="Manhattan Project"
oc new-project gemini --display-name="Gemini Project"
oc new-project bluebook --display-name="Bluebook Project"
oc new-project titan --display-name="Titan Project"

# Step 2: Make armstrong admin of apollo
oc adm policy add-role-to-user admin armstrong -n apollo

# Step 3: Make armstrong admin of gemini
oc adm policy add-role-to-user admin armstrong -n gemini

# Step 4: Give wozniak view-only access to titan
oc adm policy add-role-to-user view wozniak -n titan

# Step 5: Verify permissions
oc policy who-can get deployments -n apollo
oc policy who-can view pods -n titan
```

**Key Points:**
- `admin` role = full project administration
- `view` role = read-only access
- `edit` role = create/modify resources but not admin
- Each project is a namespace in Kubernetes

---

## **QUESTION 4: Configure Groups and Permission**

### Question:
Configure your OpenShift cluster to meet the following requirements:

- The user account **armstrong** is a member of the **commander** group
- The user account **collins** is a member of the **pilot** group
- The user account **aldrin** is a member of the **pilot** group
- Members of the **commander** group have **edit** permission in the **apollo** project
- Members of the **pilot** group have **view** permission in the **apollo** project

### Answer:

```bash
# Step 1: Create groups
oc adm groups new commander armstrong
oc adm groups new pilot collins aldrin

# Step 2: Add users to groups (already done above, but explicit form)
oc adm groups add-users commander armstrong
oc adm groups add-users pilot collins aldrin

# Step 3: Grant edit role to commander group in apollo project
oc adm policy add-role-to-group edit commander -n apollo

# Step 4: Grant view role to pilot group in apollo project
oc adm policy add-role-to-group view pilot -n apollo

# Step 5: Verify group membership and permissions
oc get groups
oc get group commander -o yaml
oc policy who-can edit deployments -n apollo
oc policy who-can get pods -n apollo
```

**Key Points:**
- Groups are used for centralized permission management
- Use `oc adm groups` for group management
- `edit` allows creating and modifying resources
- `view` is read-only access
- Groups can inherit from identity provider (HTPasswd, LDAP, etc.)

---

## **QUESTION 5: Configure Quotas**

### Question:
Configure your OpenShift cluster to use quotas in the **manhattan** project with the following requirements:

- The name of the quota is **ex280-quota**
- The amount of memory consumed across all containers may not exceed **1Gi**
- The total amount of CPU consumed across all containers may not exceed **1 full core**
- The maximum number of replication controllers does not exceed **3**
- The maximum number of pods does not exceed **3**
- The maximum number of services does not exceed **2**

### Answer:

```bash
# Step 1: Ensure manhattan project exists
oc new-project manhattan

# Step 2: Create ResourceQuota
cat <<EOF | oc apply -f -
apiVersion: v1
kind: ResourceQuota
metadata:
  name: ex280-quota
  namespace: manhattan
spec:
  hard:
    memory: "1Gi"
    cpu: "1"
    replicationcontrollers: "3"
    pods: "3"
    services: "2"
EOF

# Step 3: Verify quota creation
oc get resourcequota -n manhattan
oc describe resourcequota ex280-quota -n manhattan

# Step 4: Monitor quota usage
oc describe resourcequota ex280-quota -n manhattan --show-usage
```

**Key Points:**
- ResourceQuota limits cluster resource consumption
- Memory is in bytes (Gi, Mi, etc.)
- CPU is in cores or millicores
- Pod limits prevent resource hogging
- Quotas apply to the entire namespace

---

## **QUESTION 6: Configure the Secure Route**

### Question:
Configure and deploy a secure route. Deploy an application called **oxcart** securely in the project called **area51**.

- The application has self-signed certificate available at:
  `/C=US/ST=NC/L=Raleigh/O=RedHat/OU=RHT/CN=oxcart.apps.ocp4.example.com`
- Use newcert command to generate required tls objects, pass the subject as argument value.
- The application should be reachable at **https://oxcart.apps.ocp4.example.com**
- Application produces a valid Output.

### Answer:

```bash
# Step 1: Create area51 project
oc new-project area51

# Step 2: Generate certificate using newcert
newcert --subject=/C=US/ST=NC/L=Raleigh/O=RedHat/OU=RHT/CN=oxcart.apps.ocp4.example.com

# Step 3: This will create server.crt and server.key files

# Step 4: Create TLS secret
oc create secret tls oxcart-tls \
  --cert=server.crt \
  --key=server.key \
  -n area51

# Step 5: Deploy oxcart application
oc new-app --docker-image=quay.io/redhattraining/hello-world:latest \
  --name=oxcart -n area51

# Step 6: Wait for deployment to be ready
oc rollout status deployment/oxcart -n area51

# Step 7: Create service (if not already created by new-app)
oc expose deployment oxcart --port=8080 -n area51

# Step 8: Create secure edge route with certificate
oc create route edge oxcart \
  --service=oxcart \
  --hostname=oxcart.apps.ocp4.example.com \
  --cert=server.crt \
  --key=server.key \
  -n area51

# Step 9: Verify the route
oc get route oxcart -n area51
oc get route oxcart -n area51 -o yaml

# Step 10: Test the application
curl -k https://oxcart.apps.ocp4.example.com
```

**Key Points:**
- `newcert` generates self-signed certificates
- `route edge` terminates TLS at the edge router
- Must provide certificate and key files
- `-k` flag ignores certificate validation in curl for self-signed certs
- Use `oc new-app` for quick application deployment

---

## **QUESTION 7: Deploy the Helm Chart Application**

### Question:
Deploy the chart named **etherpad** in the project **aeti-service** from the repository:
`http://helm.ocp4.example.com/charts`

Note: You may use the telnet or nc commands to validate the deployment.

### Answer:

```bash
# Step 1: Add helm repository
helm repo add etherpad-repo http://helm.ocp4.example.com/charts
helm repo update

# Step 2: Create aeti-service project
oc new-project aeti-service

# Step 3: List available charts
helm search repo etherpad

# Step 4: Deploy etherpad chart
helm install etherpad etherpad-repo/etherpad \
  --namespace aeti-service \
  --values values.yaml  # Optional: custom values

# Step 5: Verify deployment
helm list -n aeti-service
oc get all -n aeti-service

# Step 6: Wait for pods to be ready
oc rollout status deployment -n aeti-service

# Step 7: Validate with telnet
telnet etherpad.aeti-service.svc.cluster.local 9001

# Step 8: Validate with nc
nc -zv etherpad.aeti-service.svc.cluster.local 9001

# Step 9: Get service details
oc get service -n aeti-service
oc describe service etherpad -n aeti-service
```

**Key Points:**
- Helm manages Kubernetes applications using charts
- `helm repo add` adds chart repository
- Chart names format: `repository/chart-name`
- Port 9001 is typical for Etherpad
- Use `telnet` or `nc` to test port connectivity
- Charts can include custom values files

---

## **QUESTION 8: Configure a Service Account**

### Question:
Configure a service account in the **apples** project to meet the following requirements:

- The name of the service account is **ex280sa**
- The service account allows pods to be run as **any available user id**

### Answer:

```bash
# Step 1: Create apples project
oc new-project apples

# Step 2: Create service account
oc create serviceaccount ex280sa -n apples

# Step 3: Allow the service account to run as any user ID
# Using anyuid SCC (Security Context Constraint)
oc adm policy add-scc-to-user anyuid -z ex280sa -n apples

# Step 4: Verify service account
oc get serviceaccount -n apples
oc describe serviceaccount ex280sa -n apples

# Step 5: Verify SCC binding
oc get scc anyuid -o yaml

# Alternative method (if using pod security policy):
# Create ClusterRole and ClusterRoleBinding if needed
```

**Key Points:**
- Service accounts are Kubernetes identities for pods
- SCCs (Security Context Constraints) control pod security policies
- `anyuid` allows running as any user, not just default
- `-z` specifies the service account name
- Service accounts bind to SCCs through RoleBindings

---

## **QUESTION 9: Deploy the Application**

### Question:
Deploy the application called **oranges** in the **apples** project so that the following conditions are true:

- The application uses the **ex280sa** service account
- No configuration components have been added or removed
- The application produces output.

### Answer:

```bash
# Step 1: Ensure apples project exists
oc project apples

# Step 2: Create deployment with service account
cat <<EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: oranges
  namespace: apples
spec:
  replicas: 1
  selector:
    matchLabels:
      app: oranges
  template:
    metadata:
      labels:
        app: oranges
    spec:
      serviceAccountName: ex280sa
      containers:
      - name: oranges
        image: quay.io/redhattraining/hello-world:latest
        ports:
        - containerPort: 8080
EOF

# Step 3: Create service to expose the deployment
oc expose deployment oranges --port=8080 -n apples

# Step 4: Create route for external access
oc expose service oranges -n apples

# Step 5: Wait for deployment to be ready
oc rollout status deployment/oranges -n apples

# Step 6: Verify the deployment
oc get deployment oranges -n apples
oc get pods -n apples
oc get service oranges -n apples
oc get route oranges -n apples

# Step 7: Test the application
curl http://oranges-apples.apps.ocp4.example.com
```

**Key Points:**
- `serviceAccountName` field in pod spec specifies the service account
- Service exposes pods on a stable IP and port
- Route provides external access
- No configuration changes means use default deployment strategy
- Application must produce output (return 200 with content)

---

## **QUESTION 10: Configure a Secret**

### Question:
Configure a secret in the **math** project and the name of the secret should be **magic**.

- The secret should have following key value pairs:
  - **ACCESS_STRING: ASDA142hfh-gfrhhueo-erfdk345v**

### Answer:

```bash
# Step 1: Create math project
oc new-project math

# Step 2: Create generic secret with literal value
oc create secret generic magic \
  --from-literal=ACCESS_STRING='ASDA142hfh-gfrhhueo-erfdk345v' \
  -n math

# Alternative: Create from file
echo 'ASDA142hfh-gfrhhueo-erfdk345v' > /tmp/access.txt
oc create secret generic magic \
  --from-file=ACCESS_STRING=/tmp/access.txt \
  -n math

# Step 3: Verify secret creation
oc get secret -n math
oc describe secret magic -n math
oc get secret magic -n math -o yaml

# Step 4: Decode secret (base64 is default encoding)
oc get secret magic -n math -o jsonpath='{.data.ACCESS_STRING}' | base64 -d
```

**Key Points:**
- `generic` secret type is for arbitrary data
- `--from-literal` creates from command line
- `--from-file` creates from files
- Secrets are base64 encoded (not encrypted by default)
- Key name is `ACCESS_STRING`
- Use in deployments via environment variables or volumes

---

## **QUESTION 11: Configure an Application to Use a Secret**

### Question:
Configure the application called **red** in the **math** project with the following requirements:

- The application uses the secret previously created called **magic**
- The secret defines an environment variable with name **ACCESS_STRING**
- The application output no longer displays "Sorry, application is not configured correctly"

### Answer:

```bash
# Step 1: Ensure math project is active
oc project math

# Step 2: Create deployment for red application
oc new-app --docker-image=quay.io/redhattraining/test-app:latest \
  --name=red -n math

# Step 3: Add secret as environment variable to deployment
oc set env deployment/red \
  --from=secret/magic \
  -n math

# Alternative: Using YAML manifest
cat <<EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: red
  namespace: math
spec:
  replicas: 1
  selector:
    matchLabels:
      app: red
  template:
    metadata:
      labels:
        app: red
    spec:
      containers:
      - name: red
        image: quay.io/redhattraining/test-app:latest
        env:
        - name: ACCESS_STRING
          valueFrom:
            secretKeyRef:
              name: magic
              key: ACCESS_STRING
        ports:
        - containerPort: 8080
EOF

# Step 4: Create service
oc expose deployment red --port=8080 -n math

# Step 5: Create route
oc expose service red -n math

# Step 6: Wait for rollout
oc rollout status deployment/red -n math

# Step 7: Verify environment variable is set
oc get deployment red -n math -o yaml | grep -A 10 env

# Step 8: Check pod logs to verify correct output
oc logs -f deployment/red -n math

# Step 9: Test the application
curl http://red-math.apps.ocp4.example.com
```

**Key Points:**
- `oc set env` updates environment variables on existing deployments
- `secretKeyRef` references a key from a secret
- Application receives `ACCESS_STRING` environment variable
- Secret must exist before deployment
- Pods are automatically restarted when secrets change

---

## **QUESTION 12: Configure a Network Policy**

### Question:
Configure a network policy using the **database** and **checker** projects with the following requirements:

- The database project has a network policy with the name **db-allow-mysql-conn** based on pod selector label **network.openshift.io/policy-group**
- Connections to the database project are restricted to deployments from the checker project
- The network policy is filtered by Project selector using the **team=devsecops** label
- Pod selector using the **deployment=web-mysql** label.
- The application can establish a connection to port **3306/TCP**
- You can check your work by examining the logs in the checker project.

### Answer:

```bash
# Step 1: Create database project
oc new-project database

# Step 2: Create checker project
oc new-project checker

# Step 3: Label the checker project with team=devsecops
oc label namespace checker team=devsecops --overwrite=true

# Step 4: Create network policy in database project
cat <<EOF | oc apply -f -
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-allow-mysql-conn
  namespace: database
spec:
  podSelector:
    matchLabels:
      deployment: web-mysql
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: devsecops
    ports:
    - protocol: TCP
      port: 3306
EOF

# Step 5: Deploy MySQL in database project
oc new-app --docker-image=mysql:5.7 \
  --name=mysql \
  -e MYSQL_ROOT_PASSWORD=password \
  -n database

# Step 6: Label MySQL pods with deployment=web-mysql
oc patch deployment mysql -n database -p \
  '{"spec":{"template":{"metadata":{"labels":{"deployment":"web-mysql"}}}}}'

# Step 7: Deploy web application in checker project
oc new-app --docker-image=quay.io/redhattraining/test-app:latest \
  --name=web \
  -n checker

# Step 8: Test connectivity from checker to database
# Get MySQL service name
oc get service -n database

# Create a test pod in checker project
oc run test-pod --image=mysql:5.7 -i -t --rm -- \
  mysql -h mysql.database.svc.cluster.local -u root -ppassword -e "SELECT 1"

# Step 9: Check logs in checker project
oc logs -f deployment/web -n checker

# Step 10: Verify network policy
oc get networkpolicy -n database
oc describe networkpolicy db-allow-mysql-conn -n database
```

**Key Points:**
- NetworkPolicy restricts traffic between pods
- `namespaceSelector` filters by project labels
- `podSelector` filters by pod labels
- `policyTypes: Ingress` means incoming connections only
- Port 3306 is MySQL default
- Network policies are implemented by network plugins (Calico, OVN)

---

## **QUESTION 13: Deploy an Application**

### Question:
Deploy the application called **atlas** in the **mercury** project so that the following conditions are true:

- No configuration components have been added or removed.
- The application produces output.

### Answer:

```bash
# Step 1: Create mercury project
oc new-project mercury

# Step 2: Deploy atlas application using oc new-app
oc new-app --docker-image=quay.io/redhattraining/hello-world:latest \
  --name=atlas \
  -n mercury

# Alternative: Using YAML manifest
cat <<EOF | oc apply -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: atlas
  namespace: mercury
spec:
  replicas: 1
  selector:
    matchLabels:
      app: atlas
  template:
    metadata:
      labels:
        app: atlas
    spec:
      containers:
      - name: atlas
        image: quay.io/redhattraining/hello-world:latest
        ports:
        - containerPort: 8080
EOF

# Step 3: Expose the deployment
oc expose deployment atlas --port=8080 -n mercury

# Step 4: Create a route for external access
oc expose service atlas -n mercury

# Step 5: Wait for deployment to be ready
oc rollout status deployment/atlas -n mercury

# Step 6: Verify deployment
oc get deployment atlas -n mercury
oc get pods -n mercury
oc get service atlas -n mercury
oc get route atlas -n mercury

# Step 7: Test the application
curl http://atlas-mercury.apps.ocp4.example.com

# Step 8: Check pod logs for output
oc logs deployment/atlas -n mercury
```

**Key Points:**
- `oc new-app` creates deployment, service, and image stream
- No manual configuration needed - use defaults
- Application must be accessible and return output
- Route automatically created with `oc expose service`
- Use standard hello-world image for testing

---

## **QUESTION 14: Install an Operator**

### Question:
Install the **file-integrity** operator with the following requirements:

- The operator is installed in the **openshift-file-integrity** project
- The approval strategy is **Automatic**
- Cluster monitoring is enabled for the **openshift-file-integrity** project

### Answer:

```bash
# Step 1: Create the openshift-file-integrity namespace
oc create namespace openshift-file-integrity

# Step 2: Create OperatorGroup in the namespace
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1
kind: OperatorGroup
metadata:
  name: file-integrity-operator
  namespace: openshift-file-integrity
spec:
  targetNamespaces:
  - openshift-file-integrity
EOF

# Step 3: Subscribe to file-integrity operator
cat <<EOF | oc apply -f -
apiVersion: operators.coreos.com/v1alpha1
kind: Subscription
metadata:
  name: file-integrity-operator
  namespace: openshift-file-integrity
spec:
  channel: stable
  installPlanApproval: Automatic
  name: file-integrity-operator
  source: redhat-operators
  sourceNamespace: openshift-marketplace
EOF

# Step 4: Verify operator subscription
oc get subscription -n openshift-file-integrity

# Step 5: Check install plan
oc get installplan -n openshift-file-integrity

# Step 6: Verify operator is running
oc get deployment -n openshift-file-integrity
oc get pods -n openshift-file-integrity

# Step 7: Enable cluster monitoring
oc label namespace openshift-file-integrity \
  openshift.io/cluster-monitoring=true --overwrite=true

# Step 8: Verify monitoring label
oc get namespace openshift-file-integrity --show-labels

# Step 9: Wait for operator to be ready
oc wait --for=condition=Available \
  deployment/file-integrity-operator \
  -n openshift-file-integrity --timeout=300s
```

**Key Points:**
- Operators are managed via Subscriptions in OLM (Operator Lifecycle Manager)
- `OperatorGroup` defines target namespaces
- `installPlanApproval: Automatic` auto-approves updates
- Monitoring label enables Prometheus scraping
- Channel specifies stable, beta, or alpha releases
- Source is typically `redhat-operators` from official registry

---

## **QUESTION 15: Configure Template**

### Question:
Configure your OpenShift cluster so that new projects are created with limits using the following requirements:

- The name of the limit range is **PROJECT_NAME-limits**, where **PROJECT_NAME** is the name of the project created using `oc new-project`.
- The amount of memory consumed by a single container is:
  - Minimum: **128Mi**
  - Maximum: **1Gi**
  - Default: **512Mi**
  - Default request: **256Mi**

### Answer:

```bash
# Step 1: Get the current project template
oc get -n openshift-config cm project-template -o yaml > /tmp/project-template-backup.yaml

# Step 2: Create updated project template with LimitRange
cat <<EOF | oc apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: project-template
  namespace: openshift-config
data:
  template: |-
    apiVersion: v1
    kind: ProjectRequest
    metadata:
      name: \${PROJECT_NAME}
    objects:
    - apiVersion: v1
      kind: Namespace
      metadata:
        annotations:
          openshift.io/description: \${PROJECT_DESCRIPTION}
          openshift.io/display-name: \${PROJECT_DISPLAYNAME}
        name: \${PROJECT_NAME}
    - apiVersion: v1
      kind: LimitRange
      metadata:
        name: \${PROJECT_NAME}-limits
      spec:
        limits:
        - type: Container
          min:
            memory: 128Mi
          max:
            memory: 1Gi
          default:
            memory: 512Mi
          defaultRequest:
            memory: 256Mi
    - apiVersion: v1
      kind: ResourceQuota
      metadata:
        name: \${PROJECT_NAME}-quota
      spec:
        hard:
          requests.cpu: "10"
          requests.memory: "10Gi"
EOF

# Step 3: Update cluster configuration to use the template
cat <<EOF | oc apply -f -
apiVersion: config.openshift.io/v1
kind: Project
metadata:
  name: cluster
spec:
  projectRequestTemplate:
    name: project-template
EOF

# Step 4: Verify the configuration
oc get cm project-template -n openshift-config -o yaml

# Step 5: Test by creating a new project
oc new-project test-limits

# Step 6: Verify LimitRange was created
oc get limitrange -n test-limits
oc describe limitrange test-limits-limits -n test-limits

# Step 7: Clean up test project
oc delete project test-limits
```

**Key Points:**
- Project templates are ConfigMaps in `openshift-config` namespace
- `${PROJECT_NAME}` is substituted with actual project name
- LimitRange applies per-container limits
- Default values apply when not specified in pod
- Template applies to all new projects going forward

---

## **QUESTION 16: Create a Cron Job**

### Question:
Create a cron job with the following requirements:

- The cron job name is **ipsum**
- The cron job uses the image **quay.io/redhattraining/scaling**
- The cron job runs **every minute**
- The cron job successful job history limit is **10**
- The cron job runs in a project called **lorem**

### Answer:

```bash
# Step 1: Create lorem project
oc new-project lorem

# Step 2: Create CronJob
cat <<EOF | oc apply -f -
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ipsum
  namespace: lorem
spec:
  schedule: "* * * * *"
  successfulJobsHistoryLimit: 10
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      template:
        spec:
          serviceAccountName: default
          containers:
          - name: scaling
            image: quay.io/redhattraining/scaling:latest
            imagePullPolicy: IfNotPresent
          restartPolicy: OnFailure
EOF

# Step 3: Verify CronJob creation
oc get cronjob -n lorem
oc describe cronjob ipsum -n lorem

# Step 4: Monitor job execution
oc get jobs -n lorem
oc get pods -n lorem

# Step 5: Check logs from executed jobs
oc logs -f job/ipsum-1629840120 -n lorem

# Step 6: View CronJob details
oc get cronjob ipsum -n lorem -o yaml

# Step 7: Monitor successful jobs
oc get cronjob ipsum -n lorem --show-managed-fields=false
```

**Cron Schedule Explanation:**
- Format: `minute hour day month weekday`
- `* * * * *` = every minute
- `0 * * * *` = every hour
- `0 0 * * *` = daily
- `0 0 * * 0` = weekly (Sunday)

**Key Points:**
- CronJob creates Jobs on schedule
- `successfulJobsHistoryLimit` keeps last N successful jobs
- `failedJobsHistoryLimit` keeps last N failed jobs
- Image must be available (use `imagePullPolicy: IfNotPresent`)
- `restartPolicy: OnFailure` retries failed pods
- Default service account is used unless specified

---

## **QUESTION 17: Scale the Application Manually**

### Question:
Scale an application called **hydra** in the project called **lerna**. The **hydra** application should be scaled to **five times**.

### Answer:

```bash
# Step 1: Ensure lerna project exists
oc new-project lerna

# Step 2: Deploy hydra application if not already present
oc new-app --docker-image=quay.io/redhattraining/hello-world:latest \
  --name=hydra -n lerna

# Step 3: Scale the deployment to 5 replicas
oc scale deployment hydra --replicas=5 -n lerna

# Step 4: Wait for scaling to complete
oc rollout status deployment/hydra -n lerna

# Step 5: Verify scaling
oc get deployment hydra -n lerna
oc get pods -n lerna
oc get pods -n lerna -o wide

# Alternative: Edit deployment directly
# oc edit deployment hydra -n lerna
# Change spec.replicas to 5

# Alternative: Using patch command
oc patch deployment hydra -n lerna -p '{"spec":{"replicas":5}}'

# Step 6: Monitor pods
oc get pods -l app=hydra -n lerna --watch

# Step 7: Check deployment status
oc describe deployment hydra -n lerna

# Step 8: View events
oc get events -n lerna --sort-by='.lastTimestamp'
```

**Key Points:**
- `oc scale` is the quickest way to scale
- Scaling increases/decreases replicas
- LoadBalancer/Service distributes traffic across pods
- Replicas must match available resources
- Use `--watch` to monitor scaling in real-time
- Can also use horizontal pod autoscaler (HPA) for automatic scaling

---

## **Quick Reference: Common Commands**

```bash
# Project Management
oc new-project PROJECT_NAME              # Create new project
oc project PROJECT_NAME                  # Switch to project
oc delete project PROJECT_NAME           # Delete project
oc get projects                          # List projects

# User & Group Management
oc adm policy add-cluster-role-to-user ROLE USER           # Add cluster role
oc adm policy add-role-to-user ROLE USER -n PROJECT        # Add project role
oc adm policy add-role-to-group ROLE GROUP -n PROJECT      # Add group role
oc adm groups new GROUP USER1 USER2      # Create group
oc adm groups add-users GROUP USER       # Add user to group

# Resource Management
oc create secret generic SECRET_NAME --from-literal=KEY=VALUE -n PROJECT
oc create configmap CM_NAME --from-literal=KEY=VALUE -n PROJECT
oc set env deployment/APP --from=secret/SECRET_NAME -n PROJECT

# Deployment & Scaling
oc new-app --docker-image=IMAGE --name=NAME -n PROJECT
oc scale deployment APP --replicas=N -n PROJECT
oc rollout status deployment/APP -n PROJECT
oc logs deployment/APP -n PROJECT

# Networking
oc expose deployment APP --port=PORT -n PROJECT            # Create service
oc expose service SERVICE --hostname=HOST -n PROJECT       # Create route
oc create route edge ROUTE --service=SERVICE --cert=CRT --key=KEY

# Permissions
oc policy who-can ACTION RESOURCE -n PROJECT
oc get rolebinding -n PROJECT
oc describe rolebinding NAME -n PROJECT
```

---

## **Exam Tips**

1. **Time Management**: 3 hours for 17 questions = ~10 minutes per question
2. **Read Carefully**: Check all requirements before starting
3. **Test Your Work**: Verify each deployment works
4. **Use Aliases**: Create bash aliases for frequently used commands
5. **Backup Configs**: Save manifests before modifying
6. **Know YAML**: Familiarize yourself with Kubernetes/OpenShift YAML structure
7. **Practice**: Run through labs multiple times
8. **Documentation**: Keep oc help and man pages ready
9. **Naming Convention**: Follow exact naming in requirements
10. **Order Matters**: Some tasks depend on previous ones

---

**Good Luck with your EX280 exam!**
