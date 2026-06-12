# Build a Basic Web Application on AWS

## Introduction

This project involves building a full-stack web application using AWS Amplify. It features a React frontend with user authentication, a serverless Lambda function triggered on user sign-up, and an Amazon DynamoDB database for storing user profile data. The application leverages AWS cloud services to deliver a scalable, secure user experience \- enabling sign-up, sign-in, and profile data storage without managing any server infrastructure.

## Why AWS Amplify?

AWS Amplify removes the overhead of setting up and managing backend infrastructure by providing ready-to-use services for hosting, authentication, API management, and data storage. This allows full focus on application functionality rather than configuration, making it an ideal choice for rapidly building and deploying production-ready web applications.

## What I Learned

- **Host:** Deploy a React application on the AWS global content delivery network (CDN) using AWS Amplify.  
- **Authenticate:** Implement user sign-in and sign-out functionality using Amazon Cognito via Amplify.  
- **Database:** Integrate a real-time GraphQL API backed by Amazon DynamoDB for persistent data storage.  
- **Function:** Trigger a Lambda function automatically when a user confirms their account to capture profile data.

## Tech Stack

| Service / Tool | Role |
| :---- | :---- |
| **AWS Amplify** | End-to-end hosting, CI/CD, backend provisioning |
| **AWS AppSync** | Real-time GraphQL API management |
| **AWS Lambda** | Serverless function triggered on user confirmation |
| **Amazon DynamoDB** | NoSQL database for storing user profile data |
| **Amazon Cognito** | User authentication and identity management |
| **React (Vite)** | Frontend framework |
| **Node.js & npm** | Dependency management and local development |
| **Git & GitHub** | Version control and Amplify CI/CD integration |

## Prerequisites

- Basic familiarity with AWS services (Amplify, Lambda, DynamoDB, Cognito)  
- An active AWS account with appropriate IAM permissions  
- Node.js and npm installed locally  
- Git installed and configured  
- A GitHub account with a repository for the project

## Problem Statement

Modern web applications require user authentication, data persistence, and API integration from the start. Configuring and scaling these services independently is time-consuming and error-prone. AWS Amplify solves this by providing a fully managed, integrated backend that pairs seamlessly with any React frontend.

This project implements a user profile system where users sign up, confirm their account, and have their profile data automatically stored in DynamoDB via a Lambda trigger. The pattern is applicable to any use case requiring user management \- from community platforms to internal tools.

## Architecture Diagram

## Step-by-Step Implementation

This project is divided into six tasks that must be completed in order.

### Task 1: Create a Web App

#### Overview

AWS Amplify supports a Git-based CI/CD workflow for building, deploying, and hosting single-page applications. When connected to a GitHub repository, Amplify detects the frontend framework, configures the build, and automatically redeploys on every commit.

#### Key Concepts

- **React:** A JavaScript library for building fast, performant single-page applications.  
- **Git:** A version control system for tracking changes and integrating with Amplify's deployment pipeline.

#### Implementation

**Step 1: Create a new React application**

Run the following commands to scaffold a React app using Vite. The `--template react` flag sets up JSX and a standard React project structure. Once `npm run dev` completes, a local development server starts and outputs a `localhost` URL you can open in the browser to verify the default Vite \+ React page is running.

npm create vite@latest profilesapp \-- \--template react

cd profilesapp

npm install

npm run dev

**Step 2: Push the project to GitHub**

Create a new public repository named `profilesapp` on GitHub, then run the following commands from the project root. Replace `<your-username>` with your GitHub username. After a successful push, the repository will contain the initial React scaffold on the `main` branch.

git init

git add .

git commit \-m "first commit"

git remote add origin git@github.com:\<your-username\>/profilesapp.git

git branch \-M main

git push \-u origin main

**Step 3: Install Amplify packages**

From the project root, run:

npm create amplify@latest \-y

This scaffolds a lightweight Amplify project inside the app directory, adding an `amplify/` folder with default backend configuration files. Push the changes to GitHub:

git add .

git commit \-m 'installing amplify'

git push origin main

**Step 4: Deploy with AWS Amplify**

1. Open the [AWS Amplify console](https://console.aws.amazon.com/amplify/apps) and choose **Create new app**.  
2. On the **Start building with Amplify** page, select **GitHub** as the deployment source and choose **Next**.  
3. Authenticate with GitHub when prompted. Select the `profilesapp` repository and the `main` branch, then choose **Next**.  
4. Leave the default build settings and choose **Next**.  
5. Review the configuration and choose **Save and deploy**. Amplify will build and deploy the app to an `amplifyapp.com` domain \- deployment may take up to 5 minutes.  
6. Once the build completes, select **Visit deployed URL** to confirm the live app is running.

#### Outcome

A React application is now live on the AWS global CDN, continuously deployed from GitHub on every push.

### Task 2: Build a Serverless Function

#### Overview

With the frontend deployed, the next step is configuring a Lambda function using AWS Amplify. This function is invoked after a user confirms their account (post-confirmation trigger in Amazon Cognito) and handles capturing user data.

#### Key Concepts

- **Serverless function:** Code executed on demand by a compute service, without managing underlying infrastructure.

#### Implementation

**Step 1: Set up the Amplify function**

Navigate to `profilesapp/amplify/auth/` and create a new folder named `post-confirmation`. Inside it, create two files: `resource.ts` and `handler.ts`.

The `resource.ts` file registers the function with Amplify:

// amplify/auth/post-confirmation/resource.ts

import { defineFunction } from '@aws-amplify/backend';

export const postConfirmation \= defineFunction({

  name: 'post-confirmation',

});

The `handler.ts` file defines the Lambda entry point. At this stage it simply returns the event \- it will be updated in Task 3 to write data to DynamoDB:

// amplify/auth/post-confirmation/handler.ts

import type { PostConfirmationTriggerHandler } from "aws-lambda";

export const handler: PostConfirmationTriggerHandler \= async (event) \=\> {

  return event;

};

#### Outcome

A Lambda function is defined using Amplify and ready to be connected to the data layer.

### Task 3: Create a Data Table

#### Overview

In this task, a data model is created to persist user profile data using a GraphQL API and Amazon DynamoDB. AWS IAM authorisation is used to allow the Lambda function to write to the DynamoDB table via AppSync.

#### Key Concepts

- **Amplify Gen 2 backend:** Uses full-stack TypeScript to define data models, business logic, and auth rules. Amplify provisions and deploys the correct cloud resources automatically.

#### Implementation

**Step 1: Set up Amplify Data**

Navigate to `amplify/data/resource.ts` and replace its contents with the schema below. This defines a `UserProfile` model with `email` and `profileOwner` fields. The `.authorization` rules restrict record access to the profile owner and grant the Lambda function write access via IAM.

// amplify/data/resource.ts

import { type ClientSchema, a, defineData } from "@aws-amplify/backend";

import { postConfirmation } from "../auth/post-confirmation/resource";

const schema \= a

  .schema({

    UserProfile: a

      .model({

        email: a.string(),

        profileOwner: a.string(),

      })

      .authorization((allow) \=\> \[

        allow.ownerDefinedIn("profileOwner"),

      \]),

  })

  .authorization((allow) \=\> \[allow.resource(postConfirmation)\]);

export type Schema \= ClientSchema\<typeof schema\>;

export const data \= defineData({

  schema,

  authorizationModes: {

    defaultAuthorizationMode: "apiKey",

    apiKeyAuthorizationMode: {

      expiresInDays: 30,

    },

  },

});

Deploy the schema to a sandbox environment. Once complete, the terminal confirms success and an `amplify_outputs.json` file is generated at the project root \- this file contains the backend endpoint and config values used by the frontend.

npx ampx sandbox

Generate the GraphQL client code that the Lambda function will use to write to the API:

npx ampx generate graphql-client-code \--out \<path-to-post-confirmation-handler-dir\>/graphql

Amplify creates an `amplify/auth/post-confirmation/graphql/` folder containing the generated mutations, queries, and types.

**Step 2: Update the Lambda function to write to the API**

Replace the contents of `amplify/auth/post-confirmation/handler.ts` with the following. This updated handler reads the confirmed user's email and sub from the Cognito event, then writes a new `UserProfile` record to DynamoDB via the AppSync GraphQL API using IAM auth.

// amplify/auth/post-confirmation/handler.ts

import type { PostConfirmationTriggerHandler } from "aws-lambda";

import { type Schema } from "../../data/resource";

import { Amplify } from "aws-amplify";

import { generateClient } from "aws-amplify/data";

import { env } from "$amplify/env/post-confirmation";

import { createUserProfile } from "./graphql/mutations";

Amplify.configure(

  {

    API: {

      GraphQL: {

        endpoint: env.AMPLIFY\_DATA\_GRAPHQL\_ENDPOINT,

        region: env.AWS\_REGION,

        defaultAuthMode: "iam",

      },

    },

  },

  {

    Auth: {

      credentialsProvider: {

        getCredentialsAndIdentityId: async () \=\> ({

          credentials: {

            accessKeyId: env.AWS\_ACCESS\_KEY\_ID,

            secretAccessKey: env.AWS\_SECRET\_ACCESS\_KEY,

            sessionToken: env.AWS\_SESSION\_TOKEN,

          },

        }),

        clearCredentialsAndIdentityId: () \=\> {

          /\* noop \*/

        },

      },

    },

  }

);

const client \= generateClient\<Schema\>({

  authMode: "iam",

});

export const handler: PostConfirmationTriggerHandler \= async (event) \=\> {

  await client.graphql({

    query: createUserProfile,

    variables: {

      input: {

        email: event.request.userAttributes.email,

        profileOwner: \`${event.request.userAttributes.sub}::${event.userName}\`,

      },

    },

  });

  return event;

};

#### Outcome

A DynamoDB table is provisioned via a GraphQL API, and the Lambda function is updated to write user profile data to it on confirmation.

### Task 4: Link the Serverless Function to the Web App

#### Overview

The Amplify Auth configuration is updated to register the Lambda function as a Cognito post-confirmation trigger. When a user completes sign-up, the function fires automatically and writes their profile to DynamoDB.

#### Key Concepts

- **Lambda invocation:** An event that causes a Lambda function to execute, in this case triggered by Amazon Cognito after account confirmation.

#### Implementation

**Step 1: Configure Amplify Auth**

Navigate to `amplify/auth/resource.ts` and replace its contents with the following. Adding the `triggers.postConfirmation` property registers the Lambda function as a Cognito post-confirmation trigger so it fires automatically whenever a user confirms their account.

// amplify/auth/resource.ts

import { defineAuth } from '@aws-amplify/backend';

import { postConfirmation } from './post-confirmation/resource';

export const auth \= defineAuth({

  loginWith: {

    email: true,

  },

  triggers: {

    postConfirmation

  }

});

The sandbox will detect the file change and redeploy automatically. If it is not running, start it with:

npx ampx sandbox

Once redeployment completes, the `amplify_outputs.json` file is updated to reflect the new auth configuration.

#### Outcome

Authentication is configured with the Lambda post-confirmation trigger active. User sign-up now automatically populates the DynamoDB table.

### Task 5: Add Interactivity to the Web App

#### Overview

The frontend is updated to use the Amplify UI component library, which scaffolds a complete authentication flow (sign-up, sign-in, password reset) and connects the React app to the cloud backend.

#### Key Concepts

- **Amplify libraries:** Client-side libraries for interacting with AWS backend services from a web or mobile application.

#### Implementation

**Step 1: Install Amplify libraries**

npm install aws-amplify @aws-amplify/ui-react

**Step 2: Style the app UI**

Replace the contents of `profilesapp/src/index.css` with the following:

:root {

  font-family: Inter, system-ui, Avenir, Helvetica, Arial, sans-serif;

  line-height: 1.5;

  font-weight: 400;

  color: rgba(255, 255, 255, 0.87);

  font-synthesis: none;

  text-rendering: optimizeLegibility;

  \-webkit-font-smoothing: antialiased;

  \-moz-osx-font-smoothing: grayscale;

  max-width: 1280px;

  margin: 0 auto;

  padding: 2rem;

}

.card {

  padding: 2em;

}

.read-the-docs {

  color: \#888;

}

.box:nth-child(3n \+ 1\) { grid-column: 1; }

.box:nth-child(3n \+ 2\) { grid-column: 2; }

.box:nth-child(3n \+ 3\) { grid-column: 3; }

**Step 3: Implement the UI**

Replace the contents of `profilesapp/src/main.jsx` with the following. Wrapping the app in `<Authenticator>` automatically renders the hosted sign-in and sign-up UI before the main app is shown.

// src/main.jsx

import React from "react";

import ReactDOM from "react-dom/client";

import App from "./App.jsx";

import "./index.css";

import { Authenticator } from "@aws-amplify/ui-react";

ReactDOM.createRoot(document.getElementById("root")).render(

  \<React.StrictMode\>

    \<Authenticator\>

      \<App /\>

    \</Authenticator\>

  \</React.StrictMode\>

);

Replace the contents of `profilesapp/src/App.jsx` with the following. The component configures Amplify from `amplify_outputs.json`, fetches the authenticated user's profile from DynamoDB on mount, and renders it alongside a sign-out button.

// src/App.jsx

import { useState, useEffect } from "react";

import {

  Button,

  Heading,

  Flex,

  View,

  Grid,

  Divider,

} from "@aws-amplify/ui-react";

import { useAuthenticator } from "@aws-amplify/ui-react";

import { Amplify } from "aws-amplify";

import "@aws-amplify/ui-react/styles.css";

import { generateClient } from "aws-amplify/data";

import outputs from "../amplify\_outputs.json";

Amplify.configure(outputs);

const client \= generateClient({

  authMode: "userPool",

});

export default function App() {

  const \[userprofiles, setUserProfiles\] \= useState(\[\]);

  const { signOut } \= useAuthenticator((context) \=\> \[context.user\]);

  useEffect(() \=\> {

    fetchUserProfile();

  }, \[\]);

  async function fetchUserProfile() {

    const { data: profiles } \= await client.models.UserProfile.list();

    setUserProfiles(profiles);

  }

  return (

    \<Flex

      className="App"

      justifyContent="center"

      alignItems="center"

      direction="column"

      width="70%"

      margin="0 auto"

    \>

      \<Heading level={1}\>My Profile\</Heading\>

      \<Divider /\>

      \<Grid

        margin="3rem 0"

        autoFlow="column"

        justifyContent="center"

        gap="2rem"

        alignContent="center"

      \>

        {userprofiles.map((userprofile) \=\> (

          \<Flex

            key={userprofile.id || userprofile.email}

            direction="column"

            justifyContent="center"

            alignItems="center"

            gap="2rem"

            border="1px solid \#ccc"

            padding="2rem"

            borderRadius="5%"

            className="box"

          \>

            \<View\>

              \<Heading level="3"\>{userprofile.email}\</Heading\>

            \</View\>

          \</Flex\>

        ))}

      \</Grid\>

      \<Button onClick={signOut}\>Sign Out\</Button\>

    \</Flex\>

  );

}

**Step 4: Run and verify the application**

Start the development server:

npm run dev

Open the localhost URL in the browser. You will see the Amplify UI authentication screen. Select the **Create Account** tab, enter an email address and password, and choose **Create Account**. A verification code will be sent to that email \- enter it to complete sign-up. Once signed in, the app will display the authenticated user's email address, confirming the Lambda trigger successfully wrote the profile to DynamoDB.

Push the changes to GitHub to trigger an automatic redeployment via Amplify:

git add .

git commit \-m 'displaying user profile'

git push origin main

Once the Amplify pipeline finishes, open the Amplify console and select **Visit deployed URL** to confirm the live deployment reflects all changes.

#### Outcome

The web application is fully interactive, supporting user sign-up, sign-in, and authenticated profile display powered by the Amplify backend.

### Task 6: Clean Up Resources

#### Overview

To avoid incurring unnecessary charges, all AWS resources created during this project should be deleted once the tutorial is complete.

#### Implementation

1. In the Amplify console, navigate to the `profilesapp` app, open **App settings**, and select **General settings**.  
2. Choose the Delete **app** to remove all provisioned resources.

## Challenges and Solutions

**Challenge 1: Authentication Configuration** Amplify simplifies authentication setup, but customising Cognito behaviour (such as password policies or MFA) requires additional configuration in the Amplify Console or directly in the `resource.ts` auth definition file.

**Challenge 2: Data Consistency with DynamoDB** DynamoDB is eventually consistent by default. Using AppSync real-time subscriptions helps ensure data synchronisation across all connected clients when records are created or updated.

**Challenge 3: Lambda Execution Limits** AWS Lambda imposes limits on execution time and memory. For larger data operations, function code should be optimised and memory settings adjusted through the Amplify Console or backend configuration.

## Conclusion

This project demonstrates how to build and deploy a full-stack serverless web application using AWS Amplify. Starting from a React frontend to a globally hosted app with user authentication, a Lambda trigger, a GraphQL API, and a DynamoDB data store, the entire backend was provisioned and managed through Amplify with minimal configuration overhead.

The architecture is scalable from a prototype to a production workload and illustrates core patterns used in real-world cloud-native applications \- user identity management, event-driven functions, and API-backed data persistence.

---

**Stephanie Makori** | AWS Certified Cloud Practitioner | Computer Science, Egerton University

[LinkedIn](https://www.linkedin.com/in/stephanie-makori) | [GitHub](https://github.com/stephanie-makori)  
