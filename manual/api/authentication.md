# API Authentication
SeaSearch uses HTTP Basic Auth for authentication. API requests must include the corresponding basic auth token in the header.

To generate a basic auth token, combine the username and password with a colon (e.g., aladdin:opensesame), and then base64 encode the resulting string (e.g., YWxhZGRpbjpvcGVuc2VzYW1l).

You can generate a token using the following command, for example with aladdin:opensesame:

```
echo -n 'aladdin:opensesame' | base64
YWxhZGRpbjpvcGVuc2VzYW1l
```
Note: Basic auth is not secure. If you need to access SeaSearch over the public internet, it is strongly recommended to use HTTPS (e.g., via reverse proxy such as Nginx).
```
"Authorization": "Basic YWRtaW46MTIzNDU2Nzg="
```

## Administrator User
SeaSearch uses accounts to manage API permissions. When the program starts for the first time, an administrator account must be configured through environment variables.

Here is an example of setting the administrator account via shell:
```
set ZINC_FIRST_ADMIN_USER=admin
set ZINC_FIRST_ADMIN_PASSWORD=Complexpass#123
```
!!! tip 
In most scenarios, you can use the administrator account to provide access for applications. Only when you need to integrate multiple applications with different permissions, you should create regular users.


## Regular Users
You can create/update users via the API:
```
[POST] /api/user
{ 
    "_id": "prabhat",
    "name": "Prabhat Sharma",
    "role": "admin", // or user
    "password": "Complexpass#123"
}
```
To get all users:
```
[GET] /api/user
```
To delete a user:
```
[DELETE] /api/user/${userId}
```
