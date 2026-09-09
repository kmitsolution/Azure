Here is a complete, minimal example of how to connect to Azure Cache for Redis using VS Code, a connection string, and JavaScript/TypeScript (Node.js).
## 1. Set Up Your VS Code Project
Open your VS Code terminal (Ctrl+ `) and run these commands to set up a new project and install the official Redis client:

mkdir redis-demo && cd redis-demo
npm init -y
npm install redis dotenv

## 2. Configure Your Access Keys (.env)
Create a file named .env in the root of your project. Copy your connection string from the Azure Portal (found under your Redis resource -> Authentication -> Access Keys).

# Format: redis[s]://[:password]@hostname:port
REDIS_URL=rediss://:YOUR_PRIMARY_ACCESS_KEY@YOUR_REDIS_NAME.redis.cache.windows.net:6380

(Note: Always use rediss:// with a double 's' for the secure port 6380 required by Azure).
## 3. Create the Connection Code (index.js)
Create a file named index.js and paste the following simple script to connect, write a record, and read it back:

import { createClient } from 'redis';import 'dotenv/config';
async function run() {
    // 1. Initialize the client using the connection string from your .env file
    const client = createClient({
        url: process.env.REDIS_URL
    });

    // Handle connection errors
    client.on('error', (err) => console.error('Redis Client Error', err));

    console.log('Connecting to Azure Redis...');
    await client.connect();
    console.log('Connected successfully! 🎉');

    // 2. Insert a record (Key: "user:100", Value: "John Doe") with a 60-second expiration
    await client.set('user:100', 'John Doe', { EX: 60 });
    console.log('Record inserted.');

    // 3. Read the record back
    const value = await client.get('user:100');
    console.log(`Retrieved value from Redis: "${value}"`);

    // 4. Disconnect safely
    await client.disconnect();
    console.log('Disconnected.');
}

run().catch(console.error);

## 4. Run the Code in VS Code

   1. Open your package.json file and make sure "type": "module" is added so you can use modern import syntax:
   
   {
     "name": "redis-demo",
     "type": "module",
     "dependencies": { ... }
   }
   
   2. In your VS Code terminal, execute the script:
   
   node index.js
   
   
------------------------------
## Alternative: View Data Graphically in VS Code
If you want to view your keys inside VS Code without writing code, you can use the Database Client or Redis X extensions from the VS Code Marketplace. When prompting for connection details, use your Hostname, Port (6380), and your Primary Access Key as the password with SSL enabled.
If you prefer, let me know:

* What programming language is your application using? (e.g., Python, C#/.NET, Java)
* Are you encountering any SSL/TLS handshake errors or timeout blocks from an Azure Firewall?

I can provide the specific language wrapper or network configuration steps you need.

