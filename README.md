# Fullstack Authentication Example with Next.js and NextAuth.js

This is the starter project for the fullstack tutorial with Next.js and Prisma. You can find the final version of this project in the [`final`](https://github.com/prisma/blogr-nextjs-prisma/tree/final) branch of this repo.

## Workflow

```shell
npm run dev
```

### Prisma commands

```txt
npx prisma studio
```

You need to update Prisma every time the schema file change by running the following command:

```txt
npx prisma generate
```

To create the tables in your database, you now can use the following command of the Prisma CLI:

```txt
npx prisma db push
```

## Step 6

Set up GitHub authentication with NextAuth.

### Create a new OAuth app on GitHub

1. log into your GitHub account
2. Settings/Developer Settings/OAuth Apps
3. Choose Register a new app
4. The Authorization callback URL should be the Next.js /api/auth route: http://localhost:3000/api/auth.
5. Copy the generated Client ID and Client Secret and paste them into the .env file in the root directory as the GITHUB_ID and GITHUB_SECRET env vars. Also set the NEXTAUTH_URL to the same value of the Authorization callback URL thar you configured on GitHub: http://localhost:3000/api/auth

To deploy the app later set up a new GitHub OAuth app with a production URL as in step 4.

### Shema error

In step 6 of the article, the update to the schema.prisma file cause a few issues.

First, there appears to be an extra curly bracket at the end of the file:

```prisma
model VerificationToken {
  id         Int      @id @default(autoincrement())
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])}
}
```

Unless that is some kind of Prisma schema format, it shouldn't be there.  At least my VScode editor doesn't like it and shows it in red.

That's an easy thing to remove and test.  However, either way there are huge errors when trying to update the db:

```txt
PS C:\Users\timof\repos\react\blogr-nextjs-prisma> npx prisma db push
Environment variables loaded from .env
Prisma schema loaded from schema.prisma
Error: Prisma schema validation - (get-config wasm)
Error code: P1012
error: Error validating: This line is invalid. It does not start with any known Prisma schema keyword.
  -->  schema.prisma:3
   | 
 2 | 
 3 | model Post {
 4 |   id        String  @id @default(cuid())
   | 
error: Error validating: This line is invalid. It does not start with any known Prisma schema keyword.
  -->  schema.prisma:4
   | 
 3 | model Post {
 4 |   id        String  @id @default(cuid())
 5 |   title     String
   | 
 .... 
error: Error validating: This line is invalid. It does not start with any known Prisma schema keyword.
  -->  schema.prisma:54
   | 
53 | 
54 |   @@unique([identifier, token])}
55 | 
   | 

Validation Error Count: 45
[Context: getConfig]

Prisma CLI Version : 4.14.0
```

Given the official repo for this project is about two years old, it might be a version issue.

Using the schema.prisma file from [the final branch](https://github.com/prisma/blogr-nextjs-prisma/blob/final/prisma/schema.prisma) works to update the db with one small modification.

I changed this:

```sql
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

to this:

```sql
datasource db {
  provider = "postgresql"
  url      = env("POSTGRES_URL")
}
```

### Persist authentication state across app

Change the root file _app.tsx and wrap the root component with a SessionProvider from the next-auth/react package.

## Step 7

### Add Log In functionality

The login button and some other UI components will be added to the Header.tsx file. Open the file and paste the following code into it:

```js
  const isActive: (pathname: string) => boolean = (pathname) =>
    router.pathname === pathname;

  const { data: session, status } = useSession();
```

- If no user is authenticated, a Log in button will be shown.
- If a user is authenticated, My drafts, New Post and Log out buttons will be shown.

```js
  let left = (
    <div className="left">
      <Link href="/"></a>
      </Link>
      <style jsx>{` ... `}</style>
    </div>
  );

  let right = null;

  if (status === 'loading') {
    left = (
      <div className="left">
        <Link href="/">...</Link>
        <style jsx>{` ... `}</style>
      </div>
    );
    right = (
      <div className="right">
        <p>Validating session ...</p>
        <style jsx>{`
          .right {
            margin-left: auto;
          }
        `}</style>
      </div>
    );
  }

  if (!session) {
    right = (
      <div className="right">
        <Link href="/api/auth/signin">
          <a data-active={isActive('/signup')}>Log in</a>
        </Link>
        <style jsx>{`a { text-decoration: none; ... }`}</style>
      </div>
    );
  }

  if (session) {
    left = (
      <div className="left">
        ...
      </div>
    );
    right = (
      <div className="right">
      ...
      </div>
    );
  }

  return (
    <nav>
      {left}
      {right}
      <style jsx>{`nav { display: flex; ...`}</style>
    </nav>
  );
```

NextAuth.js requires a specific route for authentication.

Create a new directory and a new file in the pages/api directory:

pages/api/auth/[...nextauth].ts

Create a new directory and API route.

Add boilerplate to configure NextAuth.js setup with GitHub OAuth credentials and the Prisma adapter.

## Step 8. Add new post functionality

touch pages/create.tsx

The backend handles the POST submitted with Next.js API routes in the pages/api directory.

touch pages/api/post/index.ts

Create a new API route to create a post.

### Invalid `prisma.post.findMany()` invocation

npx prisma studio

```txt
Message: Error in Prisma Client request: 
Invalid `prisma.post.findMany()` invocation:
Error occurred during query execution:
ConnectorError(ConnectorError { user_facing_error: None, kind: QueryError(Error { kind: Db, cause: Some(DbError { severity: "ERROR", parsed_severity: Some(Error), code: SqlState(E26000), message: "prepared statement \"s6\" does not exist", detail: None, hint: None, position: None, where_: None, schema: None, table: None, column: None, datatype: None, constraint: None, file: Some("prepare.c"), line: Some(451), routine: Some("FetchPreparedStatement") }) }), transient: false })
  
Query:
{
  "modelName": "Post",
  "operation": "findMany",
  "args": {
    "take": 100,
    "skip": 0,
    "select": {
      "id": true,
      "title": true,
      "content": true,
      "published": true,
      "author": true,
      "authorId": true
    }
  }
}
```

That findMany call has been there and working since step 5.

This fixed it.

```txt
npx prisma generate
npx prisma db push
```
