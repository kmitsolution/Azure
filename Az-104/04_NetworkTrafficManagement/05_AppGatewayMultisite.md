To configure Azure Application Gateway for multi-site hosting with two backend pools and associated listeners, follow these steps:

### 1. Create Backend Pools:

- Navigate to your Azure Application Gateway resource in the Azure portal.
- Under "Backend pools", click on "Add a backend pool".
- Create two backend pools: one for targeting `vm-1` and another for targeting `vm-2`.

### 2. Add Listeners:

- Under "Listeners", click on "Add a listener".
- Create the first listener with the hostname `www.server1.com` and associate it with the backend pool targeting `vm-1`.
- Similarly, create the second listener with the hostname `www.server2.com` and associate it with the backend pool targeting `vm-2`.

### 3. Create Backend HTTP Settings:

- Under "Backend HTTP settings", click on "Add a backend HTTP setting".
- Create two settings: `server1-http` and `server2-http`, both with port 80.

### 4. Add Rules:

- Under "Rules", click on "Add a rule".
- Create `server1-rule` for the first listener (`www.server1.com`) and associate it with the backend pool targeting `vm-1`.
- Similarly, create `server2-rule` for the second listener (`www.server2.com`) and associate it with the backend pool targeting `vm-2`.

### 5. Update Hosts File:

- On your local machine, open the `hosts` file located at `C:\Windows\System32\drivers\etc`.
- Add the following entries:
  ```
  <<AppGateway Public IP>> www.server1.com
  <<AppGateway Public IP>> www.server2.com
  ```
  Replace `<<AppGateway Public IP>>` with the public IP address of your Azure Application Gateway.

### 6. Testing:

- Open a web browser and navigate to `http://www.server1.com`.
- The request will be routed through the Azure Application Gateway to the backend pool targeting `vm-1`.
- Similarly, test `http://www.server2.com` to ensure traffic is routed to the backend pool targeting `vm-2`.

Following these steps, you've configured Azure Application Gateway for multi-site hosting with two backend pools, listeners, and rules, enabling traffic routing based on hostnames.
