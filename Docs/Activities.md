---
layout: page
title: Activities
permalink: /activities/
has_children: true
nav_order: 1
---

# Activities
Automation comes with a variety of built-in activities for core functionality. These activities handle operations such as date and time manipulation, JSON and XML parsing, and HTTP requests. This section contains helpful hints and best practices for using these activities.

## 1.1. General

### 1.1.1. Naming

Names of activities should be short and descriptive enough, and follow the rules of a capitalized sentence, e.g:

```
✅ Was the request successful?
🚫 Was The Request Successful?

✅ Retrieve important data
🚫 Perform request
🚫 Send GET request to retrieve data
```

---

### 1.1.2. Settings

There's an option to continue Workflow execution on activity failure. To do so, you should use the **Continue Workflow execution on failure** checkbox of activity.
In such a case, you should handle the errors yourself within the condition activities or any means you see fit. In many cases, this is recommended as you can then set the **Workflow Results** output property. To do this, you can add a condition block below and check for the previous activity **Succeeded** boolean property.

> [!NOTE]
> Usually, you want to use **Continue on HTTP Error Status Code** checkbox instead of **Continue Workflow execution on failure** one for HTTP Request Activities. You can also do error checking in a similar fashion with a condition block and checking the **HTTP Status Code** property of the previous HTTP Activity.

---

## 1.2. Set Variables activity

### 1.2.1. Local variables

To set one local variable, the name of the activity should specify the name of the variable. For example, if you are to set a value for a variable named **Service Group**, then the name of the activity should be **Set Service Group** instead of **Set local variable**. Besides, if you are trying to set a few variables in a bulk, use **Set local variables** naming.

---

### 1.2.2. Output variables

To set two or more variables in a bulk, go with **Set output variables** name. Otherwise, use the **Set output variable**.

---

## 1.3 Web Service Request

### 1.3.1. Naming

Web Service request activities names should start with "Request to ...", if it doesn't conflict with the way these activities are named in the existing Workflows of the same provider.

```
✅ Request to obtain data
🚫 Request for data
🚫 Obtain data
```

---

### 1.3.2. Relative URL

Use explicit **Relative URL** path instead of variables (unless necessary).

 ![wsg_3](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110391835/921c1857-ec1a-1c79-a295-088f6db1b01c)

 ![wsg_1](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110391835/e7a99e56-6cbe-17cd-bb3b-0c1eee950cb8)

Using variables for request parameters is totally OK, though.

 ![wsg_5](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110391835/c6679ec9-06ae-181d-8b7c-01dd16a00221)

---

### 1.3.3. Headers

Make sure that your **Web Service Request** activities have **Content-Type** & **Accept** headers defined.
Usually it's:

* Content Type: Application JSON
* Accept: application/json

 ![wsg_6](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110391835/170ef2b3-618f-19f2-ac20-b1cde2d56f5b)

---

## 1.4. Python activity

### 1.4.1. General mind-set

The general idea to keep in mind when you write python scripts is to keep the code simple, straightforward and more readable. The other person reading your code should be able to tell right-away what is going on and where the flow is going.

---

### 1.4.2. Code-style

Use 2-spaces indentation in favor of the more common 1-space, since it's the default format the Workflow Python-activity operates in, thus make it more usable for further modifications by different users.

Use camelCase  for names in python activities.

```python
# 🚫 bad
param_one = 1
ParamThree = 3

# ✅ good
paramOne = 1
```

For more than one input argument, use multiline statements to deconstruct and limit the slice explicitly from both start and end:

```python
(
  paramOne, 
  paramTwo, 
  paramThree
) = sys.argv[1:4]
```

> [!IMPORTANT]
> We don't use trailing commas for multiline statements like:
> * lists 
> * tuples
> * function parameters
> * etc.

Separate each import statement with a new line

```python
# 🚫 bad
import sys, json

# ✅ good
import json
import sys
```

---

### 1.4.3. Script input variables

> [!CAUTION]
> All internal XDR variable types are automatically cast into string prior to be passed to the python code!

---

#### 1.4.3.1 Input validation

You should rembemer, that all the input variables of **Execute Python Script** activities are of type string. Validate your inputs accordingly.
```python
# ✅ good
if len(input) > 0:
  output = int(input) + 5
  # ...
  
# 🚫 bad
if input > 0:
  output = input + 0.5
  # ...
```

---

### 1.4.4. Script output variables

**General rules**:

* Use *camelCase* for Property Names, the same as names in the Python activity, and avoid duplicates with input, local, or output variables.

**Special cases**:

If your script is preparing a payload for a **Web Service HTTP Request** activity, then the output payload variable should be named as **payload**.

---

### 1.4.5. Best practices

This section decribes common cases and provides templates to address them.

#### 1.4.5.1. Input validation

Since all the inputs passed to **Execute Python Script** activity would be implicitly converted to `str` type by XDR, and for the sake of transparency and readability, we've addopted the next way of input validation:

```python
import sys


(
  inputStringValue,
  inputIntValue,
  inputBoolValue,
  mandatoryInputIntValue,
) = sys.argv[1:5]


# Check if the variable is available
if len(inputStringValue) > 0:
  # do your thing

# Check if the variable is available
if len(inputIntValue) > 0:
  intValue = int(inputIntValue)
  if intValue > 5:
    newValue = intValue + 10

# Check if the variable is available
if len(inputBoolValue) > 0:
  # Note, that we need to explicitly check for the value
  if inputBoolValue.lower() == "true":
    # do your thing
  elif inputBoolValue.lower() == "false":
    # do your other thing

# For mandatory values you don't need to check for availability.
# You can just go ahead and perfom the necessary check 
if int(mandatoryInputIntValue) >= 0:
  # do your thing
# ...
```

> [!TIP]
> Use Execute Python Script activity to build **request query string** or **request payload object** if one or any combination of the following conditions are met:
> * There’s more than one optional parameter 
> * Parameter needs an extra formatting
> * Parameter needs a specific validation

---

#### 1.4.5.2. Payload building

When you build a payload for your request follow the next rules:
* set the mandatory parameters as items of the dictionary during intialization
* for each optional parameter perform a check with an `if len(parameter) >0:` check
* use function to format and/or modify the parameter value

Below is a template you can use for your query string building activities:
```python
import json
import sys


(    
  mandatoryParameter,  
  optionalParameter,
  extraFormattedParameter,
  optionalExtraFormattedParameter
) = sys.argv[1:5]

# 
def formatExtraFormattedParameter(value):
  return value


def formatOptionalExtraFormattedParameter(value):
  return value


payload = {
  "mandatoryParameter": mandatoryParameter,
  "extraFormattedParamter": formatExtraFormattedParameter(extraFormattedParameter)
}


if len(optionalParameter) > 0:
  payload["optionalParameter"] = optionalParameter
if len(optionalExtraFormattedParameter) > 0:
  payload["optionalExtraFormattedParameter"] = formatOptionalExtraFormattedParameter(optionalParameter)

payload = json.dumps(payload)
```

---

#### 1.4.5.3. Query string building

When you build a query string for your request follow the next rules:
* set the mandatory parameters as members of the list during initializatioin
* for each optional parameter perform a check with an `if len(parameter) >0:` check
* use function to format and/or modify the parameter value

> [!NOTE]
> All variable types would be casted to string before they are passed to **Execute Python Script** activity.

Below is a template you can use for your payload building activities:
```python
import sys


(    
  mandatoryParameter,  
  optionalParameter,
  extraFormattedParameter,
  optionalExtraFormattedParameter
) = sys.argv[1:5]


def formatExtraFormattedParameter(value):
  return value


def formatOptionalExtraFormattedParameter(value):
  return value


args = [
  f"mandatoryParameter={mandatoryParameter}",
  f"extraFormattedParameter={formatExtraFormattedParameter(extraFormattedParameter)}"
]

if len(optionalParameter) > 0:
  args.append(f"optionalParameter={optionalParameter}")
if len(optionalExtraFormattedParameter) > 0:
  parameterValue = formatOptionalExtraFormattedParameter(optionalExtraFormattedParameter)
  args.append(f"optionalExtraFormattedParameter={parameterValue}")

queryString = '&'.join(args)

if len(queryString) > 0:
  queryString = f"?{queryString}"
```

---

#### 1.4.5.4. Pagination

When working with Workflow atomics, if the API endpoint you are working with supports pagination. To implement it you'll usually need two variables:

1. a cursor or a starting index, e.g.: `startAt`, `start`, etc.
2. an amount limiter, e.g.: `limit`, `maxResults`, `maxCount`, etc.

For each of those parameters we provide validations abinding to the next rule: "You can't start from negative value and you cant have zero results". See the below code sample for details:
```python
import sys


(
  startAt,
  maxResults
) = sys.argv[1:3]


args = []
if int(startAt) >= 0:
  args.append(f"startAt={startAt}")
if int(maxResults) > 0:
  args.append(f"maxResults={maxResults}")

# ...

```

> [!NOTE]
> Naive type casting to **int** is safe, because integer values are always required to have value in XDR.

---

#### 1.4.5.5. URL Encoding

If the API endpoint supports query parameters with whitespaces and special symbols, it's recommended to inmplemet URL Encoding for reliability and robustness purposes.

```python
import sys
from urllib.parse import quote


( 
  optionalParameter
) = sys.argv[1]

args = []

if len(optionalParameter) > 0:
  args.append(f"optionalParameter={quote(optionalParameter)}")

queryString = '&'.join(args)

if len(queryString) > 0:
  queryString = f"?{queryString}"
```

---

#### 1.4.5.6. Processing an inconsistent response data

Some APIs return a different response structure depending on different input data. In this case, the standard `JSONPath activity` may result in an error when trying to extract missing parameters. For such cases, you can use a Python activity that extracts existing keys or returns empty entities.

> [!NOTE]
> Implement a Python activity only in case when extracted values are optional. Otherwise, use [JSONPath Query activity](#17-jsonpath-query-activity) by default.

```python
import json
import sys

eventDetails = sys.argv[1]
eventDetails = json.loads(eventDetails)

optionalParameter = eventDetails.get("optionalParameter", "")
nestedOptionalParameter = eventDetails.get("dictionaryOptionalParameter", {}).get("nestedOptionalParameter", "")
```

---

## 1.5. Condition activity

Treat condition ativities as **questions**(1) and condition branches as concrete **answers**(2).

 ![wsg_7](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110391835/aceb7a12-6817-16d2-be8b-921189e5d5c5)

---

### 1.5.1. HTTP request condition activity in atomics

> [!IMPORTANT]
> Besides rule described in [1.5. Condition activity](#15-condition-activity) we have an exception for **Web Service** `HTTP Request` activity inside of atomics.

> [!NOTE]
> You don't need to tick the "Continue Workflow Execution On Failure" checkbox in the **Web Service** activity except neccesary cases.

![Wrong scenario for Web Service activity](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110190757/da9ee11c-1b18-195b-a586-bf1cec1828dc)


For such activity as **Web Service** `HTTP Request` you should name condition branches below as **"`Successful status code`/Success"** and **"Anything else"**. After that check the `Status Code` for each of condition branches instead of `Succeeded` variable. 

> [!NOTE]
> Naming of the activity should be as described in **[Web Service Request Naming](#131-naming)** chapter.

![Condition branches naming](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110190757/57e877fc-009c-1f9f-8b39-19303dad3e11)
![Status code check](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110190757/56a3e12c-a03a-161b-badb-9d37725d1a15)


Names of the condition branches should be descriptive, as is said in [1.1.1. Naming](#111-naming), and follow the rules of a capitalized sentence, e.g:


For positive scenario:
```
✅ 200/Success
✅ 201/Created
🚫 200 - Success
🚫 200/Yes
🚫 Status code: 200
```

For negative scenario:
```
✅ Anything else
🚫 Anything Else
```

---

#### 1.5.1.1. Exceptions in condition activity

There are some additional cases which you should be aware of. See below for details.

1. __There might be no need to have a positive scenario__
   
For example, if you won't had any activity such as **Set output variables** or any sort of data extraction, you can skip the positive scenario and create branch just for negative one to end the workflow in case of fail, break the loop inside of the atomic, etc.
In this case you should name condition branch as in example and still check the status code:

```
✅ No
🚫 Anything Else
```
![Just negative scenario for request activity](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110190757/cc907ea7-181c-119d-b607-3f31d3d7101c)

> [!NOTE]
> There might be rare case, we you should create branch only for positive scenario, which also take a place (in case if HTTP request unablee to fail even for negative scenario). Then you should go also with **"`Successful status code`/Success"** naming.

2. __There might be more actions__
   
In case you have several results, add them by status code and action, otherwise name condition branch as above:
![More than 2 scenario](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110190757/ffcac879-11d5-16cb-9331-11f86171d951)


---

## 1.6. For Each activity

When naming the **For each** activity always specify entity name which will be used in a loop.

 ![wsg_8](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/110391835/fcb68a6b-5813-15cc-a160-f162c7f2198c)

---

## 1.7. JSONPath Query activity

### 1.7.1. Best practicies
If you use **JSONPath Query** activity to extract data, make sure to:
1. Tick on the **Continue Workflow Execution On Failure** checkbox
1. Create a condition activity below to check for success of the operation
1. (In general cases) Fail the Workflow if the extraction was unsuccessful

![wsg_9](https://github.com/CiscoSecurity/sxo-05-security-workflows-private/assets/120575516/0076ff57-5836-1613-9bef-3da6de1fa651)

>[!NOTE]
> If API returns variable response structure, implement the extraction using `Python activity`. Check [Processing an inconsistent response data](#1156-processing-an-inconsistent-response-data) section for details.

---

## 1.8. Conventions

### 1.8.1. Conditional activities to check the environment

When you have a condition, that checks the type of the environment a Workflow is running on, there are three main and two additional types of them:

> [!CAUTION]
> **INT** and **TEST** should ***ONLY*** be present in the ***DEVELOPMENT*** versions of your Workflows.

| Abbreviation | Type | Name                        | Condition        | Value       |
| :----------- | :--- | :-------------------------- | :--------------- | :---------- |
| **NAM**      | main | North America               | equals           | `prod-name` |
| **EU**       | main | Europe                      | equals           | `prod-eu`   |
| **APCJ**     | main | Asia, Pasific, China, Japan | equals           | `prod-apjc` |
| **INT**      | dev  | -                           | matches wildcard | `*int*`     |
| **TEST**     | dev  | -                           | matches wildcard | `*test*`    |

---
