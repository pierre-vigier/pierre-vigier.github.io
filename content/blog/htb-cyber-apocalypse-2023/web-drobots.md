---
title: "[web] Drobots"
date: 2023-03-24T13:40:27+08:00
summary: an easy web ctf
draft: false
---
This challenge exposes a login page with a form as a home page.

The login form is calling an endpoint `/api/login`, with 2 parameters `username` and `password`.
We do have access to the source code, which is a python Flask.

The route `/api/login` is reaching the following code:

```python
@api.route('/login', methods=['POST'])
def apiLogin():
    if not request.is_json:
        return response('Invalid JSON!'), 400
    
    data = request.get_json()
    username = data.get('username', '')
    password = data.get('password', '')
    
    if not username or not password:
        return response('All fields are required!'), 401
    
    user = login(username, password)
    
    if user:
        session['auth'] = user
        return response('Success'), 200
        
    return response('Invalid credentials!'), 403
```

Let's explore the login function by itself:

```python
def login(username, password):
    # We should update our code base and use techniques like parameterization to avoid SQL Injection
    user = query_db(f'SELECT password FROM users WHERE username = "{username}" AND password = "{password}" ', one=True)

    if user:
        token = createJWT(username)
        return token
    else:
        return False
```

We see here that the paremters are passed as is in the SQL query, which leads to potential SQL injection, database used is mysql.

We can then craft a username to bypass the password check. By sending the username `admin" # `  and any value for password.

![flag](img/web-drobots-flag.png)
