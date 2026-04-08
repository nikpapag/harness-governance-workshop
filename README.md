# Lab 1 - Deploy to Prod

### Summary: Setup a Pipeline, to deploy the backend to a production deployment

### Outcome: A Deployment Pipeline

### Learning Objective(s):

- Understand the Harness Pipeline setup (including services and environments)
- Utilise complex deployment strategies to reduce blast radius of a release 

**Steps**
1. From the left hand menu, navigate to **Projects** → **Select the project available**\
   ![](https://lh7-us.googleusercontent.com/docsz/AD_4nXfhuMykMsIHl-7FjliWssHc0uwRpdLdrnq7GkGAI0g6UBZM69F1zpQ8ZA8N_vMqjpoGFYFR_weJk7OtOGGa2bksIaS6BlktwytmuJ1THM3e8O6tDT18HYWwFyGUye8ubsrHBChI8ORrCQ88JcKWpLjQ0DsXDS0NSZrkfZ4RUQ?key=cRG2cvp_PHVW0KG2Gq6Y_A)

1) From the left hand side menu select **Pipelines**

2) Click **+ Create a Pipeline**, enter the following values, then click **Start**
   
| Field                                  | Value            | Notes
| -------------------------------------- | ---------------- | ------------------------------------------------------------------------------------------ |
| **Name**                                   |<pre>`workshop`</pre>|                                                                                            |
| How do you want to setup your pipeline |Inline| This indicates that Harness (rather than Git) will be the source of truth for the pipeline |

3. From Pipeline Studio, Click **Add Stage** and select **Deploy** as the Stage Type

4. Enter the following values and click on **Set Up Stage**


| Input           | Value          | Notes |
| --------------- | -------------- | ----- |
| **Stage Name**      |<pre>`backend`</pre>|       |
| Deployment Type |Kubernetes|       |

5. Configure the **backend** Stage with the following\
   **Service**

- Click **- Select -**  on the **"Select Service"** input box and configure as follows:


| Input | Value       | Notes |
| ----- | ----------- | ----- |
| **Name**  |<pre>`backend`</pre>|       |

- Click **Apply Selected** and then click **Continue** to go to the **"Environment"** tab

**Environment**

The target infrastructure has been pre-created for us. The application will be deployed to a k8s cluster on the given namespace  

- Click **- Select -** on the **"Specify Environment"** input box 

- Select **prod** environment and click **"Apply Selected"**

| Input | Value | Notes                                                             |
| ----- | ----- | ----------------------------------------------------------------- |
| Name  |prod| Make sure to select the environment and infrastructure definition |

- Click **- Select -** on the **"Specify Infrastructure"** input box

-  From the dropdown select k8s

| Input | Value | Notes |
| ----- | ----- | ----- |
| Name  |k8s|       |

- Click **Continue** 

**Execution**

- Select **Canary**  and click on **Use Strategy**

6. Click **Save** and then click **Run** to execute the pipeline with the following inputs. As a bonus, save your inputs as an Input Set before executing (see below)

# Lab 2 - Security

**Summary:** Since the pipeline deploys to prod it is a requirement to have a security validation step prior to the deploy prod stage. 

<img width="979" height="445" alt="image" src="https://github.com/user-attachments/assets/b3603575-acb3-45c3-a8b5-7731d06b36ba" />

**Learning Objective(s):**

- Understand how governance plays a role in the path to production

- DevSecOps practices can be easily achieved

**Steps**

1. In the existing pipeline, **before** backend stage click on the plus icon to add a new step
2. From the available stage types select **Security**
3. Setup the stage

| Input | Value | Notes |
| ----- | ----- | ----- |
| **Stage Name**  | <pre>`Security Validation`</pre> |       |
| **Repository**  | <pre>`harness repo`</pre> | |


4. For the infrastructure select **Cloud**
5. Select **Add Step**
6. From the step library select the **Run** step
7. Set the command

| Input | Value | Notes |
| ----- | ----- | ----- |
| **Command**  | <pre>`echo "security"`</pre> | To speed up the process we just run an echo command |   


7. Apply changes
8. Save the pipeline and run it

# Lab 3 - DevSecOps

**Summary:** Our security team has implemented orchestration of **Fortify** and **OWASP** scans for our code in a reusable form **(templates)**. In order to improve our security posture they have also added policies to enforce us to include those templates

<img width="981" height="467" alt="image" src="https://github.com/user-attachments/assets/262d9967-0e2f-425e-b020-2974df4fb33a" />


**Learning Objective(s):**

- Understand how governance plays a role in the path to production

- Reusable templates make developer’s life easier

- DevSecOps practices can be easily achieved

**Steps**

1. In the existing pipeline, within the Security stage click on the plus icon to add a new step

2. Select use template\
   ![](https://lh7-us.googleusercontent.com/docsz/AD_4nXeC5rTVxlk7DeZeU_cINwcKo6Nf2wVW9brQ9MiCEfppJwmU-uH3QcNZ53qTxhur57KeySksoDBg9EqjhgKOgAEDKon6iNz9cFxozBe9VZssV-t77VNo6t1zPUvm6e2NOZJDKncxd9c2GM4HE-h-L4cIOl4u6Uqx_azoKchMdg?key=cRG2cvp_PHVW0KG2Gq6Y_A)

3. Select **DevX Fortify Scan** 

4. Name the step **Fortify**

5. In the existing pipeline, within the Security stage click on the plus icon to add a new step

6. Select use template

7. Select **OWASP**

8. Name the step **OWASP**

9. Click **Save** and then click **Run** to execute the pipeline with the following inputs

| Input       | Value | Notes |
| ----------- | ----- | ----- |
| Branch Name |main|       |

After the **Security** stage is complete, go to the **Security Tests** tab to see the deduplicated, normalized and prioritized list of vulnerabilities discovered across your scanners.

# Lab 4 - Governance/Policy as Code

**Create a Policy to block critical CVEs**

1. From the secondary menu, select **Project Settings** and select **Policies**

2. Select the **Policies** tab 

3. click **+ New Policy**

| Input | Value | Notes |
| ----- | ----- | ----- |
| **Name**  | <pre>`Runtime CVEs`</pre> |       |

4. Set the rego to the following and click **Save**

```
package securityTests

import future.keywords.in
import future.keywords.if

# Define a set of Output Variables that are denied
deny_list :=([
# Fail if there any new HIGHs that are not exempted, and outside of our established baseline
  {
    "name": "NEW_CRITICAL", "value": 0, "operator": ">"
  },
  {
    "name": "CRITICAL", "value": 0, "operator": ">"
  },
# Optionally define more Output Variables here
#  {
#    "name": "HIGH", "value": 0, "operator": ">"
#  }
])
  
#### DO NOT CHANGE THE FOLLOWING SCRIPT ####

deny[msg] {
  item =  deny_list_violations[i][j]
  variable := item.variable
  violation = item.violation
  
  msg := sprintf("Criticals ['%s'] matches the following item found on the deny list '%s'", [variable.name, violation])
}

deny_list_violations[violations] {
    input[i].name == "output"
    output_variables := input[i].outcome.outputVariables
    ov_name := object.keys(output_variables)[j]
    
    violations := [x | 
        x := {
            "variable": {"name": ov_name}, 
            "violation": deny_list[k]
        }
        deny_compare(ov_name, output_variables[ov_name], deny_list[k])
        count(x.violation) > 0
    ] 
    count(violations) > 0 
 
}

deny_compare(ov_name, ov_value, rule) := true if {
  ov_name == rule.name
  num_compare(to_number(ov_value), rule.operator, rule.value)
} 

str_compare(a, "==", b) := a == b
str_compare(a, "!", b) := a != b
str_compare(a, "~", b) := regex.match(b, a)
str_compare(a, null, b) := a == b if { b != null}
str_compare(a, null, null) := true

num_compare(a, "==", b) := a == b
num_compare(a, "<=", b) := a <= b
num_compare(a, ">=", b) := a >= b
num_compare(a, "<", b) := a < b
num_compare(a, ">", b) := a > b
num_compare(a, null, b) := a == b if { b != null}
num_compare(a, null, null) := true

semver_compare(a, "<=", b) := semver.compare(b, a) <= 0 
semver_compare(a, "<", b) := semver.compare(b, a) < 0
semver_compare(a, "==", b) := semver.compare(b, a) == 0 
semver_compare(a, ">", b) := semver.compare(b, a) > 0
semver_compare(a, ">=", b) := semver.compare(b, a) >= 0 
semver_compare(a, "!", b) := semver.compare(b, a) == 0
semver_compare(a, "~", b) := regex.match(b, a)
semver_compare(a, null, b) := semver.compare(b, a) == 0 if { b != null}
semver_compare(a, null, null) := true
```

5.  Save the Policy
6.  Select the **Policy Sets** tab
7.  Click **+ New Policy Set** and configure as follows

| Input                      | Value                     | Notes |
| -------------------------- | ------------------------- | ----- |
| **Name**                       |<pre>`Criticals Not Allowed`</pre>|       |
| **Entity Type**                |Custom|       |
| **Event Evaluation**           |On Step|       |
| Policy Evaluation Criteria |                           |       |
| Policy to Evaluate         |Runtime CVEs|       |

7. For the new policy set, toggle the **Enforced** button

**Add Policy to Security Template**

1. From the secondary menu, select **Project Settings** and select **Templates**
2. Select the **OWASP** template
3. Click on the **Open in Template Studio**
4. Under the **Advanced** section expand **Policy Enforcement**
5. Add/Modify Policy Sets
6. Select the **Security Test Policies**
7. Click **Apply** amd **Save**
8. Navigate to the pipeline and **run** it again

**Observe the failed OWASP step**
1. Pipeline execution fails with a policy violation

<img width="1805" height="457" alt="image" src="https://github.com/user-attachments/assets/3f0ff7c0-7f5b-4266-b377-b05229485a32" />

# Lab 6 - Exemption handling
It seems our policy worked and blocked our Criticals!

Lets loop in our security teams, to exempt these critical

1. Navigate to your "Vulnerabilties" tab and perform the following action for each one of the criticals
2. Select the CVE then Request Exemption
3. Review the request. Add details if you would like, or just select "Create Request."
4. Approve the exemption at a project level
   
<img width="1128" height="751" alt="image" src="https://github.com/user-attachments/assets/0ec87cca-379a-4c4f-924e-668c7d893634" />

5. Rerun the pipeline and observe that this time the policy will not block the promotion


 1. From the module menu select **Security Testing Orchestration**
 2. Then from the left hand side menu select **Exemptions** and see the full audit history of all exemptions

> [!IMPORTANT]
> Harness has full granular role based access allowing security engineers to manage/approve exemptions



# Lab 7 - Automated Change Approval

### Summary: Setup an automated CAB process based on risk validation

### Outcome: Fully embedded pre production change control

### Learning Objective(s):
- Understand the Harness contextual policy engine

**Steps**

1. From the secondary menu, select **Project Settings** and select **Templates**
2. Open the **Automated CAB** template and observe the flow
3. Identify the release payload as evidence and the conditional execution/promotion based on the governance validation rules

**Enforce the automated change**

1. From the secondary menu, select **Project Settings** and select **Policies**
2. Select **Policy Sets**
3. Toggle the policy enforcement for the policy **automated_cab**
4. Navigate to the **workshop** pipeline and try to run it


**Enforced Change Approval**
<img width="973" height="345" alt="image" src="https://github.com/user-attachments/assets/cbd71d38-fcaa-4d20-9248-ca2c7fa09b97" />

**Flow control**
1. In the pipeline in edit mode, after the backend deploy stage click on **+Add Stage**
2. Select **Use template**
3. From the list of available template pick the **Automated CAB** template
4. Give it a name

> [!IMPORTANT]
> Notice the use of the change approval board after we deploy to prod

<img width="541" height="170" alt="image" src="https://github.com/user-attachments/assets/d6ffffdd-43df-40e8-9fce-01372c07e3d2" />

5. **Save** and **Run** the pipeline
6. Pipeline fails with a different message as it detected the flow issue 

<img width="981" height="346" alt="image" src="https://github.com/user-attachments/assets/f0a2db9a-1099-4b9e-a825-ca496f0cb30b" />

1. In the pipeline in edit mode, drag and drop the CAB stage before the deploy stage
2. **Save** and **Run** the pipeline
3. While the pipeline is running review the CAB step output
