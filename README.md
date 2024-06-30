# Fordpass via HomeAssistant
This is a hacky but effective technique to affect your ford with Home Assistant.  I have a Bronco.

##FordPass API Signup Process

#Flow 1
1. First, you MUST have a FordPass account. If you don't, you will need to do that first and that's outside of the scope for these instructions. Start here
2. You need a developer account as well https://developer.ford.com/apis
3. Follow the API instructions there to get the key things you need for this integration
   - ClientId
   - ClientSecret
   - VehicleId
   - ApplicationId

Your nodes are going to look like this:
![image](https://github.com/nicholasmparker/FordpassApiDForHa/assets/36079497/b4d1ef4d-c05c-4336-8502-981dda1e7e4d)


1) Injection node to run every 25 minutes to get a refresh token
2) Function Node (code below) to set up refresh token call

```
msg.method = "POST";
msg.url = "https://dah2vb2cprod.b2clogin.com/914d88b1-3523-4bf6-9be4-1b96b4f6f919/oauth2/v2.0/token?p=B2C_1A_signup_signin_common";
msg.headers = {
    "Content-Type": "application/x-www-form-urlencoded"
};

// Create the form-encoded payload
msg.payload = Object.entries({
    "grant_type": "refresh_token",
    "client_id": "YOUR CLIENT ID",
    "client_secret": "YOUR SECRET",
    "refresh_token": "YOUR REFRESH TOKEN"
}).map(([key, value]) => `${encodeURIComponent(key)}=${encodeURIComponent(value)}`).join('&');

return msg;
```
5. An HTTP post node, all blank
6. Function node to store your token
```
flow.set('access_token', msg.payload.access_token);
console.log(msg.payload.access_token);
return msg;
```

#Flow 2
1. MQTT in Node. I have my topic set to home/ford/engine/start
2. Function Node (code below) to set up refresh token call
```
msg.method = "POST";
msg.url = "https://api.mps.ford.com/api/fordconnect/v1/vehicles/{YOURVEHICLEID}/startEngine";
msg.headers = {
    "Authorization": "Bearer " + flow.get('access_token'),
    "Application-Id": "YOUR APPLICATION ID",
    "Content-Type": "application/json"
};
msg.payload = {};
return msg;
```
3. An HTTP post node, all blank
