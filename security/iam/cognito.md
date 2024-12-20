---
icon: user-magnifying-glass
coverY: 0
---

# Cognito

> **Cognito User Pool** provides a high-level, easy-to-use service which handles **authentication** — it verifies the identity of a user (for example, by using username/password, social logins, or multi-factor authentication).

## CUP (who you are :green\_heart:).

#### 1. **Cognito User Pool** 🏊‍♂️

* **Cognito User Pool** is fully managed serverless auth service
  * charged for both **underlying storage** and **API calls**
  * Charges based on the number of **monthly active users (MAUs)** that interact with your User Pool. A **monthly active user (MAU)** is defined as a user who has logged in, signed up, or otherwise interacted with your User Pool within a given month.
  * **Free Tier**: 50,000 MAUs per month for CUP  and a generous amount of API calls (for most use cases).
  * **Paid Tier**: Beyond the free tier, you **pay for each additional MAU** and for API calls (sign-ups, logins, etc.). Pricing varies by region, but it typically starts at around **$0.0055 per MAU** (this can change based on region and the specific type of interaction).&#x20;
  * Sign-up, sign-in, and multi-factor authentication (MFA) API calls are usually priced at $0.03 per 1,000 API calls.
* Social Login via Google, Fb, SAML, OpenID Connect or **developer-authenticated identities**
* JWT support to validate user access to backend stuff
* MFA support
* Integrates with ALB (acts as reverse proxy)and API GW (offload validation)
  * ALB is configured with action to Authenticate Cognito and has HTTPS listener
* Blocks users whos creds been compromised
* It provides functionality for user **sign-up**, **sign-in**, and **account recovery**.
* It also supports storing **custom user attributes** (e.g., `phone number`, `address`, `roles`, etc.) that you can define at the time of pool creation.

**Key points**:

* The **underlying storage** (which is a database) is **managed by AWS**. You don't have direct access to it.
* Instead of querying the database directly (like you would with DynamoDB), you interact with **Cognito APIs** to manage users, retrieve attributes, and handle authentication.

#### 2. **Accessing User Data with Cognito** 🔑

* **Cognito APIs** let you interact with the user data stored in the **User Pool**. For example, you can:
  * **Get user details** (via `adminGetUser` or `getUser`).
  * **Modify attributes** (via `adminUpdateUserAttributes`).
  * **Authenticate users** (via `initiateAuth` for login).

```javascript
const AWS = require('aws-sdk');
const cognito = new AWS.CognitoIdentityServiceProvider();

// Get user attributes using getUser API
const params = {
  AccessToken: 'ACCESS_TOKEN_HERE' // Get from user after login
};

cognito.getUser(params, function(err, data) {
  if (err) {
    console.log("Error fetching user attributes", err);
  } else {
    console.log("User attributes:", data);
  }
});
```

* You would use the `AccessToken` or `IdToken` to authenticate and query the user information from the **Cognito User Pool**.

#### 3. **Custom Attributes** ✏️

You can **add custom attributes** (like `roles`, `preferences`, etc.) to user profiles in Cognito. These are stored in Cognito's user data store, but you cannot query this database directly like you would with DynamoDB.

```javascript
//adding custom attributes
const params = {
  UserPoolId: 'userPoolId',
  Username: 'user@example.com',
  UserAttributes: [
    {
      Name: 'custom:role',
      Value: 'admin'
    }
  ]
};

cognito.adminUpdateUserAttributes(params, (err, data) => {
  if (err) {
    console.error("Error updating user attributes:", err);
  } else {
    console.log("User attributes updated successfully:", data);
  }
});
```

#### 4. **Lambda Triggers with Cognito** 🧑‍💻

* You can also set up **Lambda triggers** in Cognito to perform actions at various stages of the user authentication flow, such as during **sign-up**, **pre-sign-up validation**, or **post-confirmation**.
* These Lambda triggers allow you to modify user data during the flow or invoke additional logic.

**Post Confirmation** trigger:

```javascript
 // welcome email or log user info
exports.handler = async (event) => {
  console.log("User confirmed:", event);
  return event;
};
```

#### 5. **Cognito vs DynamoDB** 🤔

* **DynamoDB** is a general-purpose NoSQL database, and you have complete control over the schema, structure, and queries.
* **Cognito**, on the other hand, is a fully managed authentication service that abstracts away much of the complexity of managing user identities, password policies, and authentication flows.

If you want **direct access to user data** and want to control the database and schema yourself, **DynamoDB** is a better choice. However, if you're just managing user authentication and don't need to manage the underlying database directly, **Cognito** is the better option.



## Integrations



## CIP (what you can do :raccoon: :octagonal\_sign:).

> CUP- handles **authorization** — after a user has been authenticated (either through Cognito User Pools or another identity provider), it provides them with **temporary AWS credentials** (via AWS STS) to access specific AWS resources, based on the **IAM policies** attached to their identity.





Summary Flow:

#### **Cognito User Pool (CUP)** - Provides the actual **JWT token (ID token and access token)**.

* The **CUP** (Cognito User Pool) is responsible for managing user authentication. When a user signs in (e.g., through Google, Facebook, or any other identity provider), the **CUP** issues a **JWT token** (specifically an ID token and an access token).

#### 2. **Cognito Identity Pool (CIP)** - Validates the JWT and exchanges it for temporary credentials.

* The **CIP**   takes the JWT token from the **CUP** (or any external identity provider) and validates it.
* Once the **CIP** verifies the user's identity, it **exchanges** the JWT token for **temporary AWS credentials** via **AWS STS (Security Token Service)**.
  * These temporary credentials can be used to access AWS resources, like S3, DynamoDB, etc.
  * The credentials consist of an **Access Key ID**, **Secret Access Key**, and **Session Token**, all of which are short-lived.

#### 3. **Accessing AWS Resources (e.g., S3)**:

* With the temporary credentials provided by the **CIP**, the authenticated user can access specific resources, such as **S3 buckets**, based on the **IAM policy** attached to the **CIP roles**.
  * In the IAM policy, you can use the `sub` (subject) from the **JWT token** in a `Condition` block to grant or deny access to specific resources.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::your-bucket-name/*",
      "Condition": {
        "StringEquals": {
          "cognito-identity.amazonaws.com:sub": "${cognito-identity.amazonaws.com:sub}"
        }
      }
    }
  ]
}

```

#### Prerequisites :heavy\_multiplication\_x: :

1.  **Create IAM Role + Policy**:

    * The IAM policy will specify what resources the authenticated user can access (e.g., S3, DynamoDB, etc.).
    * You will likely use a variable like `cognito-identity.amazonaws.com:sub` (which is the unique identifier of the user in Cognito) in the policy to restrict access to user-specific resources.


2. **Create CIP**&#x20;

* The **CIP** will be used to authenticate the users via the **CUP** (User Pool) or any external identity provider (e.g., Google).
* After the user is authenticated, the **CIP** will issue temporary AWS credentials using **STS** for accessing AWS resources (e.g., S3).

3.  #### **Attach the Role to the Identity Pool**:

    * <mark style="background-color:yellow;">In your</mark> <mark style="background-color:yellow;"></mark><mark style="background-color:yellow;">**CIP**</mark><mark style="background-color:yellow;">, you will specify the IAM role that authenticated users will assume.</mark> This is where you attach the IAM role that has the policy to access specific resources like S3.
    * The role should be attached under the **authenticated role** in your **CIP** configuration.







<details>

<summary>JWT Token in Details <span data-gb-custom-inline data-tag="emoji" data-code="1f575">🕵️</span></summary>

**Types of Tokens:**

* **ID Token (`id_token`)**: Contains identity information about the authenticated user, like name and email. This token is typically used by the frontend to personalize the experience for the user.
* **Access Token (`access_token`)**: Contains information about the permissions and roles granted to the user. This is typically used for authorization to access AWS services or custom backend services.
* **Refresh Token (`refresh_token`)**: Used to obtain new ID and Access tokens when the old ones expire.

</details>







#### Useful Links

[https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-post-authentication.html](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-post-authentication.html)

