# GitHub API Notes

## What do we need?

1. How does authorizing a user work?
2. Make sure that the user is actually part of the MLH Fellowship.
3. How do we get a list of repos the user in involved with.
   - For each of those repos we need to get:
     - PRs
     - Commits
     - Issues
     - Code Reviews

## Implementation Details

### Authorization

[Source](https://developer.github.com/v3/#authentication)

To authorize user we need to run the following command:

```bash
curl -H "Authorization: token OAUTH-TOKEN" https://api.github.com
```

Note that this command, will probably be ran by the backend server.

So now the question is, how do we get `OAUTH-TOKEN`?

Well [this page](https://developer.github.com/apps/building-oauth-apps/authorizing-oauth-apps/#web-application-flow) seems to detail that.

TLDR:

1. The user clicks some sort of sign in with GitHub button.
2. A get request is made:

```http
GET https://github.com/login/oauth/authorize
```

With the following query parameters

```yaml
client_id: The client ID you received from GitHub when you registered

redirect_uri: >
  The URL in your application where users will be sent after authorization.
  We can use something like localhost:8080/oauth/callback etc, etc.

scope: >
  A space-delimited list of scopes.
  I think we can get away without specifying any scopes.
  Since our application is only going to read data from the user.
  Needs further investigation
```

The result of this is an `access code`.
The frontend should then send this code to backend.

The backend can then exchange this `access code` for an `access token` by sending a `POST` request.

```http
POST https://github.com/login/oauth/access_token
```

with the following query parameters:

```yaml
client_id: The client ID you received from GitHub for your GitHub App.

client_secret: >
  The client secret you received from GitHub for your GitHub App.
  Note this is the reason were making this request from the backend.
  There's no way to securely store the client secret on the frontend.

code: The access code received in the step before this one.
```

Now the backend has an auth token that it can use to grab data!

#### Authorization Recap

1. Frontend makes request to get access code.
2. Frontend sends access code to backend.
3. Backend uses the access code to get an access token.
4. Backend uses this access token to make calls to the GitHub API.

##### Authorization Routes

In order for this to work the backend needs to add one OAUTH route.
See the `spec.yaml` file for more information.

The frontend should be able to respond to something akin to /oauth/callback where it should do the following:

1.  Send a post request as specified in the `spec.yaml`.
2.  Once it receives a response from the backend it should do the following:
    - If a 2xx response:
      - Redirect back to application
      - Store the username in a cookie or session (idk too much about this part)
    - If not a 2xx response:
      - Display an error screen.
