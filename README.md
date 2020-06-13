# GraphQL Prisma MongoDB Boilerplate

A boilerplate GraphQL server with:

- [GraphQL Yoga](https://github.com/prisma/graphql-yoga) for a Scalable GraphQL server (based on Apollo Server & Express)
- [Prisma](https://www.prisma.io/) for a GraphQL database ORM (running on MySQL)
- [GraphQL Playground](https://github.com/prisma/graphql-playground)
- [Nodemon](https://github.com/remy/nodemon) to automatically restart the server when code changes are made
- [Dotenv](https://github.com/motdotla/dotenv) to handle environment variables
- JWT Authentication built in
- Example models for Users and Posts

---

## Requirements

- Node, NPM or *Yarn*
- Prisma CLI installed globally - `npm i -g prisma`
- Docker and Docker Compose (installed and running)

---

## Getting Started (local dev)

```sh
# --- Prisma Server --- #

# 1. Start up a local Prisma Server and DB via Docker
docker-compose up -d

# 2. Copy `.env.example` to `.env` and add your new DB endpoint and update the keys
cp .env.example .env && nano .env

# 3. Deploy the data models to your local Prisma server
npm prisma-deploy-dev

# --- GraphQL Server --- #

# 4. Install dependencies
npm install

# 5. Start GraphQL server (runs playground on http://localhost:4000)
npm start
```

---

## Deploying to Production

```sh
# --- Prisma Server --- #

# 1. Setup a Production Prisma Server
#    - Heroku - https://bit.ly/2THPbYk
#    - Digital Ocean - https://bit.ly/2JdM713
#    - AWS EC2 - https://bit.ly/2TEcces
#    - AWS Fargate - https://bit.ly/2FdFGqV

# 2. Add the respective Environment variables to .env.prod
cp .env.example .env.prod && nano .env.prod

# 3. Deploy the data models to the production Prisma server
yarn prisma-deploy-prod

# --- GraphQL Server --- #

# 4. Deploy the GraphQL Server to
#    - Now.sh - https://bit.ly/2HvNXrC
#    - Heroku - https://bit.ly/2FcIbtl
#    - OR Serverless AWS Lambda (does not support subscriptions):

# First setup an AWS IAM user to get your keys - https://bit.ly/2TJDcsq
export AWS_ACCESS_KEY_ID=<secret>
export AWS_SECRET_ACCESS_KEY=<secret>
yarn graphql-deploy-serverless

# 5. Add the environment variables from .env.prod
#    - For AWS Lambda:
#      - Login to the AWS console > Lambda and browse to the function/s
#      - Add each of the variables from `.env` in the 'Environment variables' section of each function
```

---

## Commands

|| Command | Description |
| --- | --- | --- |
| **Prisma Server** |
|| `yarn prisma-start` | **Starts** the local Prisma server and database |
|| `yarn prisma-stop` | **Stops** the local Prisma server and database |
|| `yarn prisma-deploy-dev` | **Deploys** Prisma server to using `.env` |
|| `yarn prisma-deploy-prod` | **Deploys** Prisma server to using `.env.prod` |
| **GraphQL Server** |
|| `yarn graphql-start` | **Starts** the GraphQL-Yoga server |
|| `yarn graphql-start-debug` | **Starts** the GraphQL-Yoga server in debug mode |
|| `yarn graphql-deploy-serverless` | **Deploys** GraphQL server to Serverless (Lambda) |
| **Tests** |
|| `yarn test` | **Tests** GraphQL server |

---

## File Structure

```
/prisma/                # Prisma database service configuration
    /generated/         # Generated files from the `prisma generate` command
    /datamodel.graphql  # Defines the database structure (written in GraphQL SDL)
    /prisma.yml         # The root configuration file for your Prisma database service
    /seed.graphql       # The database seed
/src/                   # The source files for your GraphQL server
    /Helpers/           # Helper functions
    /Resolvers/         # All Resolvers for the application schema
        /Models/        # Updates to data model queries (eg. adding a property to a response)
        /Mutation/      # Custom mutations (eg. sign up, login...)
        /Query/         # Custom queries (eg. me)
        /Subscription/  # Define the data that can be subscribed to
    /index.js           # The entry point for your GraphQL server
    /schema.graphql     # The schema defining the API exposed to client applications
```

---

## Available Queries & Mutations

### User / Authentication

<details><summary>Sign Up</summary>
<p>

```
mutation {
  signup(
    email: "zeus@examples.com"
    password: "secret42"
    firstName: "Zeus"
    lastName: "Zellot"
  ) {
    token
    user {
      id
      firstName
      lastName
      email
    }
  }
}
```
</p>
</details>

<details><summary>Login</summary>
<p>

```
mutation {
  login(
    email: "zeus@examples.com"
    password: "secret42"
  ) {
    token
    user {
      email
    }
  }
}
```
</p>
</details>

<details><summary>Update My Account</summary>
<p>

```
mutation {
  updateUser(
    email: "zeus@example.com"
    firstName: "Jane"
    lastName: "Doe"
    password: "123Abc123Abc"
  ) {
    id
    firstName
    lastName
    email
  }
}
```
</p>
</details>

<details><summary>Me</summary>
<p>

*Requires `"Authorization": "Bearer ..."` header*

```
query {
  me {
    id
    firstName
    lastName
    email
    posts {
      title
      published
    }
  }
}
```
</p>
</details>

<details><summary>Forgot Password (send password reset email)</summary>
<p>

```
mutation {
  forgotPassword(
    email: "alice@prisma.io"
  ) {
    message
  }
}
```
</p>
</details>

<details><summary>Reset Password (using token from email)</summary>
<p>

```
mutation {
  resetPassword(
    resetToken: "v0db287br2atjvfsrc4wl"
    password: "secret42"
  ) {
    token
    user {
      id
      name
      email
    }
  }
}
```
</p>
</details>

<details><summary>Verify Email (using token from email)</summary>
<p>

```
mutation {
  verifyEmail(
    emailVerifiedToken: "v0db287br2atjvfsrc4wl"
  ) {
    id
    name
    email
  }
}
```
</p>
</details>

### Post

<details><summary>Create a (draft) Post</summary>
<p>

*Requires `"Authorization": "Bearer ..."` header*

```
mutation {
  createDraft(
    title: "New Draft Post"
    content: "Hello new post that's not posted yet"
  ) {
    id
    title
    content
    author {
      firstName
      lastName
    }
  }
}
```
</p>
</details>

<details><summary>List Posts - List of All Published Posts</summary>
<p>

```
query {
  listPosts {
    id
    title
    content
    author {
      firstName
      lastName
    }
  }
}
```
</p>
</details>

<details><summary>List Posts - List of My Drafted Posts</summary>
<p>

*Requires `"Authorization": "Bearer ..."` header*

```
query {
  listDraftPosts {
    id
    title
    content
    author {
      firstName
      lastName
    }
  }
}
```
</p>
</details>

<details><summary>Read a Post</summary>
<p>

```
query {
  post(id: "cjt74m7fy06at0b18gk07p7ks") {
    id
    title
    content
    author {
      firstName
      lastName
    }
  }
}
```
</p>
</details>

<details><summary>Update a Post (one of my draft Posts to be Published)</summary>
<p>

*Requires `"Authorization": "Bearer ..."` header*

```
mutation {
  publish(id: "cjt6q05lh2qi70b45gkux1gfv") {
    id
    title
    content
    published
    author {
      firstName
      lastName
    }
  }
}
```
</p>
</details>

<details><summary>Delete a Post (one that I own)</summary>
<p>

*Requires `"Authorization": "Bearer ..."` header*

```
mutation {
  deletePost(id: "cjt6q6des2shk0b45gtusvd9d") {
    id
    title
  }
}
```
</p>
</details>

<details><summary>Subscribe to new Posts (not supported by Lambda yet)</summary>
<p>

```
subscription {
  feedSubscription {
    id
    title
  }
}
```
</p>
</details>
