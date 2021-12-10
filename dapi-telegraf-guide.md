### How to connect a Telegraf to DAPI:

1. Get a utility that can make custom requests to web API. One example is [Postman](https://www.postman.com/downloads/).
2. Create a Basic authentication token for your AIMS user. This is done by converting a string in this format: `"email@address.com:account_password"` to Base64, which can be done using a number of online resources, or by executing the following in Dev Console in most browsers: `btoa("email@address.com:account_password")` (replace `email@address.com` and `account_password` with our actual user email and password).
Throughout this guide, you'll need to replace strings in angle brackets with corresponding values. Save the resulting token value for later reference as `BASIC_AUTH_TOKEN`.
3. In a browser, open the AIMS environment you want to connect Telegraf to and navigate to `Configuration -> Agents` in the top-right menu (cog icon).
4. Find 'Environment API address' and copy the GUID (hexadecimal value with dashes) at the end.
E.g. if the address is saying `https://api.aimsinnovation.com/api/environments/12345678-abcd-0000-cdef-123456790ab/`, the part you want is `12345678-abcd-0000-cdef-123456790ab`.
This is your environment ID – save it for later reference as `ENVIRONMENT_ID`.
5. Create an endpoint. You need to pick a name for the system where the data is going to be shown (save it as `SYSTEM_NAME`), and then send an API request to `https://api.aimsinnovation.com/dapi/v1/config/endpoints` with an auth header like this:
```
Authorization: Basic <BASIC_AUTH_TOKEN>
```
and a payload like this:
```
{
  "environmentId":  "<ENVIRONMENT_ID>",
  "systemName":  "<SYSTEM_NAME>",
  "formatCode":  "telegraf"
}
```
Payload should be sent with Content-Type header `application/json`, which you can specify in the UI of most web API apps.
The response will be a JSON object. Copy its "id" value, which is a random 16-character string, e.g. `QWERtyui12345678`. This is your endpoint token, save it as `DAPI_TOKEN`.

6. In Telegraf config, find or add a section for HTTP output. Edit it to look like this:
```
[[outputs.http]]
  url = "https://dapi-eu-n.aimsinnovation.com/endpoints/<DAPI_TOKEN>"
  timeout = "10s"
  method = "POST"
  data_format = "json"
  precision = "1ns"
  json_timestamp_units = "1ns"
  [outputs.http.headers]
    Content-Type = "application/json; charset=utf-8"
```
7. In the same config, find section named `[agent]`, and update or create the following value:
```
  precision = "1ns"
```
8. Start Telegraf service.
