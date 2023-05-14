# Getting Started

Here's how to get started.

### 1. Import Connectivly Client

=== "Python"

    Install the [`connectivly-python`](https://github.com/connectivly/connectivly-python) package from pip. 

    ``` bash
    pip install connectivly-python
    ```

    ``` python title="app.py"
    from connectivly import ConnectivlyClient
    connectivly = ConnectivlyClient('API-KEY', 'BASE-URL')
    ```

### 2. Create Callback

Now create a callback URL for verifying a user's OAuth request. This route
MUST be authenticated (the user must be logged in to be able to access it.)

The user will be redirected to this route when they initiate the OAuth flow. It will
will verify the user, approve (or deny) their request, and then redirect back to 
Connectivly to finish the OAuth dance.

=== "Python"

    This example uses Flask:

    ``` python title="app.py"
    @app.route('/connectivly')
    @login_required
    def connectivly():
        token = request.args["token"]
        session = connectivly.get_login_session(token)

        # Logic to validate session: validate user, scopes, permissions

        approval = connectivly.approve_login_session(token)
        return redirect(approval['redirect_uri'])
    ```

### 3. Protect your endpoints

After a user completes the OAuth flow, we want to be able to accept these 
tokens for authentication.

The `token` is a signed JWT that contains information about the user, app,
and allowed scopes. You may use this to further make sure the user is allowed
to access the resource.

=== "Python"

    ``` python title="app.py"
    @app.route('/protected')
    def protected():
        auth_header = request.headers.get('Authorization')
        token = connectivly.validate_token(auth_header)

        if token is None:
            return 'Unauthorized', 401

        # The token is a dict with information about the user (id, scopes, etc).
        # Use it to ensure they have permission to access this route.

        return 'Hello world!'
    ```