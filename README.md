Blind App on Nillion. Here's a structured guide to help you get started:

### Step 1: Install the Nillion SDK
1. **Install nilup**:
   ```bash
   curl https://nilup.nilogy.xyz/install.sh | bash
   ```

2. **Confirm installation**:
   ```bash
   nilup -V
   ```

3. **Install the latest Nillion SDK**:
   ```bash
   nilup install latest
   nilup use latest
   nilup init
   ```

4. **Optionally enable telemetry**:
   ```bash
   nilup instrumentation enable --wallet <your-eth-wallet-address>
   ```

5. **Confirm Nillion tool installation**:
   ```bash
   nillion -V
   ```

### Step 2: Create a Nada Project and Write a Nada Program
1. **Create a new Nada project**:
   ```bash
   mkdir quickstart
   cd quickstart
   nada init nada_quickstart_programs
   ```

2. **Set up a Python virtual environment**:
   ```bash
   cd nada_quickstart_programs
   python3 -m venv .venv
   source .venv/bin/activate
   pip install nada-dsl
   ```

3. **Write your first Nada program**:
   - Create a new file `src/secret_addition.py`:
     ```bash
     touch src/secret_addition.py
     ```

   - Copy the following code into `secret_addition.py`:
     ```python
     from nada_dsl import *

     def nada_main():
         party1 = Party(name="Party1")
         my_int1 = SecretInteger(Input(name="my_int1", party=party1))
         my_int2 = SecretInteger(Input(name="my_int2", party=party1))
         new_int = my_int1 + my_int2
         return [Output(new_int, "my_output", party1)]
     ```

4. **Configure the Nada project**:
   - Add the program to `nada-project.toml`:
     ```toml
     [[programs]]
     path = "src/secret_addition.py"
     name = "secret_addition"
     prime_size = 128
     ```

5. **Compile, run, and test the program**:
   - Build the program:
     ```bash
     nada build
     ```

   - Generate test values:
     ```bash
     nada generate-test --test-name secret_addition_test secret_addition
     ```

   - Run the program with test values:
     ```bash
     nada run secret_addition_test
     ```

   - Modify the test values in `tests/secret_addition_test.yaml`:
     ```yaml
     ---
     program: secret_addition
     inputs:
        my_int1:
           SecretInteger: "3"
        my_int2:
           SecretInteger: "3"
     expected_outputs:
        my_output:
           SecretInteger: "6"
     ```

   - Re-run the test:
     ```bash
     nada test secret_addition_test
     ```

### Step 3: Build a Blind App with the cra-nillion Starter Repo
1. **Clone the CRA-Nillion starter repo**:
   ```bash
   git clone https://github.com/NillionNetwork/cra-nillion.git
   ```

2. **Install dependencies and run the app**:
   ```bash
   cd cra-nillion
   npm i
   npm start
   ```

3. **Spin up a local Nillion devnet**:
   ```bash
   nillion-devnet --seed my-seed
   ```

4. **Create and update the `.env` file**:
   - Copy the `.env.example` file:
     ```bash
     cp .env.example .env
     ```

   - Update the `.env` file with values from the `nillion-devnet` environment file.

### Step 4: Connect Your Nada Program to the Blind App
1. **Copy the program files**:
   ```bash
   cp nada_quickstart_programs/src/secret_addition.py cra-nillion/public/programs
   cp nada_quickstart_programs/target/secret_addition.nada.bin cra-nillion/public/programs
   ```

2. **Update the `ComputePage.tsx` file**:
   - Set `programName` to `secret_addition`:
     ```javascript
     const programName = 'secret_addition';
     ```

### Step 5: Run the Blind Computation Demo
1. **Navigate to the Blind Computation Demo page**:
   ```url
   http://localhost:8080/compute
   ```

2. **Follow the steps on the page to test the blind computation flow**.

### Step 6: Deploy Your Blind App to the Nillion Testnet
1. **Follow the deployment instructions on the Nillion website**.



---

# Deploy Your Blind App to the Nillion Testnet

Your blind app is currently running locally against the Nillion Devnet. Let's configure environment variables to point at the Nillion Testnet so anyone can interact with your blind app once it's deployed.

### Step 1: Update Your `.env` File and Test Locally

Update your `.env` values to point at the Nillion Testnet:

```env
REACT_APP_API_BASE_PATH=/nilchain-proxy

# Testnet .env file
REACT_APP_NILLION_CLUSTER_ID=b13880d3-dde8-4a75-a171-8a1a9d985e6c
REACT_APP_NILLION_BOOTNODE_WEBSOCKET=/dns/node-1.testnet-photon.nillion-network.nilogy.xyz/tcp/14211/wss/p2p/12D3KooWCfFYAb77NCjEk711e9BVe2E6mrasPZTtAjJAPtVAdbye
REACT_APP_NILLION_NILCHAIN_JSON_RPC=http://65.109.222.111:26657
REACT_APP_NILLION_NILCHAIN_REST_API=http://65.109.222.111:26657
REACT_APP_NILLION_NILCHAIN_CHAIN_ID=nillion-chain-testnet-1
REACT_APP_NILLION_NILCHAIN_PRIVATE_KEY=YOUR-NIL-FUNDED-PRIVATE-KEY
```

**Note:** The `REACT_APP_NILLION_NILCHAIN_PRIVATE_KEY` value should correspond to an address you've funded with Testnet NIL.

### Step 2: Create a Nillion Wallet and Get the Private Key

Follow the [Creating a Nillion Wallet](https://docs.nillion.network/wallet-guide) guide to create your Nillion wallet. Use the "Sign up with Google" option because Keplr only exposes the private key of wallets created this way.

**Steps:**
1. Create your Nillion wallet with Google.
2. Extract the private key for use in the `.env` file.

### Step 3: Fund the Nillion Wallet Address

Follow the [Nillion Faucet Guide](https://docs.nillion.network/faucet-guide) to get Testnet NIL to fund your Nillion wallet address. This will allow your app to pay for operations.

### Step 4: Test the Configuration Locally

Test your blind app locally to ensure the full blind computation flow is working as expected with the updated environment variables.

### Step 5: Set Headers and Set Up Proxy for Nilchain

The JavaScript Client uses browser web-workers. To make your app cross-origin isolated, you'll need to set COOP and COEP headers in your `webpack.config.js`.

**Update `webpack.config.js`:**

```javascript
module.exports = {
  devServer: {
    static: {
      directory: path.join(__dirname, 'public'),
    },
    headers: {
      'Cross-Origin-Embedder-Policy': 'require-corp',
      'Cross-Origin-Opener-Policy': 'same-origin',
    },
    hot: true,
    client: {
      overlay: false,
    },
    historyApiFallback: true,
    proxy: [
      {
        context: ['/nilchain-proxy'],
        target: process.env.REACT_APP_NILLION_NILCHAIN_JSON_RPC,
        pathRewrite: { '^/nilchain-proxy': '' },
        changeOrigin: true,
        secure: false,
      },
    ],
  },
};
```

**References:**
- [Enabling SharedArrayBuffer](https://developer.chrome.com/blog/enabling-shared-array-buffer/)
- [COOP and COEP](https://web.dev/articles/coop-coep)
- [Webpack Dev Server](https://webpack.js.org/configuration/dev-server/)

### Step 6: Commit Your Project to GitHub

1. Commit your repo to GitHub.
2. Tag your GitHub repo with `nillion-nada` so the Nillion community can find it.

### Step 7: Host Your Blind App with Vercel

1. Follow the [Vercel Getting Started Guide](https://vercel.com/docs/getting-started-with-vercel/import) to import your GitHub project to Vercel.
2. Add all Testnet environment variables as described in the [Vercel Environment Variables Guide](https://vercel.com/docs/projects/environment-variables).

**Set up the `vercel.json` file with headers and proxy rewrites:**

```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/static-build",
      "config": {
        "distDir": "dist"
      }
    }
  ],
  "rewrites": [
    {
      "source": "/nilchain-proxy",
      "destination": "http://65.109.222.111:26657"
    }
  ],
  "headers": [
    {
      "source": "/(.*)",
      "headers": [
        {
          "key": "Cross-Origin-Opener-Policy",
          "value": "same-origin"
        },
        {
          "key": "Cross-Origin-Embedder-Policy",
          "value": "require-corp"
        },
        {
          "key": "Access-Control-Allow-Origin",
          "value": "*"
        },
        {
          "key": "Access-Control-Allow-Methods",
          "value": "GET, POST, PUT, DELETE, PATCH, OPTIONS"
        },
        {
          "key": "Access-Control-Allow-Headers",
          "value": "X-Requested-With, content-type, Authorization"
        }
      ]
    }
  ],
  "env": {
    "REACT_APP_API_BASE_PATH": "/nilchain-proxy"
  }
}
```

### Step 8: Share Your Live Link

Share your live link on Nillion's GitHub Discussions Show and Tell Forum.

### Step 9: Keep Exploring

Congratulations on completing the Nillion Developer Quickstart and deploying your blind app to the Nillion testnet. Keep exploring and building by:

- Reading about Nillion concepts and the Nada Language.
- Learning how to interact with and manage programs, secrets, and permissions on the Nillion Network with Nillion Client.
- Challenging yourself to create a page that solves the millionaires problem.

---

This guide provides a comprehensive overview of the steps needed to deploy your blind app to the Nillion Testnet and share it with the community.
