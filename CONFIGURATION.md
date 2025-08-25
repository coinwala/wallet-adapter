# Configuration Guide

This guide covers all configuration options and settings available for the `@hyperlink/wallet-adapter` library.

## Table of Contents

- [Basic Configuration](#basic-configuration)
- [Advanced Configuration](#advanced-configuration)
- [Environment Configuration](#environment-configuration)
- [Security Configuration](#security-configuration)
- [UI Configuration](#ui-configuration)
- [Network Configuration](#network-configuration)
- [Mobile Configuration](#mobile-configuration)

## Basic Configuration

### Required Parameters

#### clientId

**Type:** `string`
**Required:** Yes
**Description:** Unique identifier for your dApp, obtained from the HyperLink team.

```typescript
const config = {
  clientId: "your-unique-client-id-here",
};
```

**Note:** Contact the HyperLink team to obtain your client ID. This is used for allowlist validation and security purposes.

#### title

**Type:** `string`
**Required:** Yes
**Description:** The name of your dApp that will be displayed in the wallet interface.

```typescript
const config = {
  title: "My Awesome dApp",
};
```

#### theme

**Type:** `'system' | 'light' | 'dark'`
**Required:** Yes
**Description:** The visual theme for the wallet interface.

```typescript
const config = {
  theme: "system", // Automatically follows system preference
};

// Or explicitly set:
const config = {
  theme: "light", // Always use light theme
};

const config = {
  theme: "dark", // Always use dark theme
};
```

**Theme Options:**

- `'system'` - Automatically follows the user's system theme preference
- `'light'` - Forces light theme regardless of system preference
- `'dark'` - Forces dark theme regardless of system preference

## Advanced Configuration

### Platform Support

#### installedOnDesktop

**Type:** `boolean`
**Required:** No
**Default:** `true`
**Description:** Whether the wallet is available on desktop platforms.

```typescript
const config = {
  installedOnDesktop: true, // Enable desktop support
};
```

#### installedOnIos

**Type:** `boolean`
**Required:** No
**Default:** `true`
**Description:** Whether the wallet is available on iOS devices.

```typescript
const config = {
  installedOnIos: true, // Enable iOS support
};
```

#### installedOnAndroid

**Type:** `boolean`
**Required:** No
**Default:** `false`
**Description:** Whether the wallet is available on Android devices.

```typescript
const config = {
  installedOnAndroid: true, // Enable Android support
};
```

### UI Customization

#### hideDraggableWidget

**Type:** `boolean`
**Required:** No
**Default:** `false`
**Description:** Whether to hide the draggable wallet widget.

```typescript
const config = {
  hideDraggableWidget: true, // Hide the draggable widget
};
```

#### hideWalletOnboard

**Type:** `boolean`
**Required:** No
**Default:** `false`
**Description:** Whether to hide the wallet onboarding interface.

```typescript
const config = {
  hideWalletOnboard: true, // Hide wallet onboarding
};
```

## Environment Configuration

### Build Environment

The library automatically detects and configures the build environment. Currently, only production is supported:

```typescript
import { HYPERLINK_BUILD_ENV } from "@hyperlink/wallet-adapter";

// The library automatically uses production environment
const buildEnv = HYPERLINK_BUILD_ENV.PRODUCTION;
```

**Available Environments:**

- `PRODUCTION` - Production environment (currently the only supported option)
- `STAGING` - Staging environment (not yet supported)
- `DEVELOPMENT` - Development environment (not yet supported)
- `ADAPTER` - Adapter environment (not yet supported)
- `LOCAL` - Local environment (not yet supported)

## Security Configuration

### Allowlist Protection

The library automatically validates your domain against the allowlist using your client ID. This ensures that only authorized domains can use your wallet integration.

**Automatic Validation:**

- Domain validation happens on wallet initialization
- Uses your client ID and the current domain
- Automatically disconnects if validation fails

**Security Features:**

- Client ID validation
- Domain allowlist checking
- Secure session management
- UUID-based session identification

## Network Configuration

### Wallet Adapter Network

#### walletAdapterNetwork

**Type:** `WalletAdapterNetwork.Mainnet | WalletAdapterNetwork.Devnet`
**Required:** No
**Default:** `WalletAdapterNetwork.Mainnet`
**Description:** The Solana network to use for wallet operations.

```typescript
import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";

const config = {
  walletAdapterNetwork: WalletAdapterNetwork.Mainnet, // Use mainnet
};

// Or use devnet for testing:
const config = {
  walletAdapterNetwork: WalletAdapterNetwork.Devnet, // Use devnet
};
```

### RPC Configuration

When registering the wallet, you must provide an RPC URL:

```typescript
const unregister = registerHyperLinkWallet({
  clientId: "your-client-id",
  title: "Your dApp",
  theme: "system",
  rpcUrl: "https://api.mainnet-beta.solana.com", // Required RPC URL
  walletAdapterNetwork: WalletAdapterNetwork.Mainnet,
});
```

**Recommended RPC Endpoints:**

- **Mainnet:** `https://api.mainnet-beta.solana.com`
- **Devnet:** `https://api.devnet.solana.com`
- **Testnet:** `https://api.testnet.solana.com`

## Mobile Configuration

### Mobile Detection

The library automatically detects mobile devices and adjusts behavior accordingly:

**Automatic Detection:**

- iOS device detection
- Android device detection
- WebView detection
- PWA mode detection

**Mobile Optimizations:**

- Automatic iframe mode for certain mobile scenarios
- Touch-optimized interface
- Responsive design
- PWA compatibility

### PWA Support

For Progressive Web Apps, the library automatically enables iframe mode:

```typescript
// The library automatically detects PWA mode
// No additional configuration needed
```

## Complete Configuration Example

Here's a complete configuration example with all options:

```typescript
import { registerHyperLinkWallet } from "@hyperlink/wallet-adapter";
import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";

const walletConfig = {
  // Required parameters
  clientId: "your-client-id-from-hyperlink",
  title: "My Solana dApp",
  theme: "system",

  // Platform support
  installedOnDesktop: true,
  installedOnIos: true,
  installedOnAndroid: false,

  // UI customization
  hideDraggableWidget: false,
  hideWalletOnboard: false,

  // Network configuration
  walletAdapterNetwork: WalletAdapterNetwork.Mainnet,

  // RPC configuration (required for registration)
  rpcUrl: "https://api.mainnet-beta.solana.com",
};

// Register the wallet
const unregister = registerHyperLinkWallet(walletConfig);

// Later, unregister when needed
unregister();
```

## Configuration Best Practices

### 1. Environment-Specific Configuration

```typescript
const isProduction = process.env.NODE_ENV === "production";

const config = {
  clientId: process.env.HYPERLINK_CLIENT_ID,
  title: "My dApp",
  theme: "system",
  walletAdapterNetwork: isProduction
    ? WalletAdapterNetwork.Mainnet
    : WalletAdapterNetwork.Devnet,
  rpcUrl: isProduction
    ? "https://api.mainnet-beta.solana.com"
    : "https://api.devnet.solana.com",
};
```

### 2. Dynamic Configuration

```typescript
const getWalletConfig = (userTheme: string) => ({
  clientId: "your-client-id",
  title: "My dApp",
  theme: userTheme as "system" | "light" | "dark",
  installedOnDesktop: true,
  installedOnIos: true,
  installedOnAndroid: false,
});
```

### 3. Configuration Validation

```typescript
const validateConfig = (config: any) => {
  if (!config.clientId) {
    throw new Error("clientId is required");
  }
  if (!config.title) {
    throw new Error("title is required");
  }
  if (!["system", "light", "dark"].includes(config.theme)) {
    throw new Error("Invalid theme value");
  }
  return config;
};

const config = validateConfig({
  clientId: "your-client-id",
  title: "My dApp",
  theme: "system",
});
```

## Troubleshooting Configuration Issues

### Common Issues

1. **Missing clientId**

   - Error: "clientId is required"
   - Solution: Contact HyperLink team to obtain your client ID

2. **Invalid theme value**

   - Error: "Invalid theme value"
   - Solution: Use only 'system', 'light', or 'dark'

3. **Network configuration mismatch**

   - Error: Network-related errors
   - Solution: Ensure walletAdapterNetwork matches your RPC URL

4. **Domain not allowlisted**

   - Error: Wallet disconnects automatically
   - Solution: Ensure your domain is properly configured with HyperLink

### Configuration Validation

The library automatically validates your configuration and will throw descriptive errors for invalid configurations. Always wrap your configuration in try-catch blocks:

```typescript
try {
  const unregister = registerHyperLinkWallet(config);
} catch (error) {
  console.error("Wallet configuration error:", error.message);
  // Handle configuration error
}
```

````

### 4. EXAMPLES.md

```markdown
# Examples

This document provides comprehensive examples of how to use the `@hyperlink/wallet-adapter` library in various scenarios.

## Table of Contents

- [Basic Integration](#basic-integration)
- [React Integration](#react-integration)
- [Transaction Examples](#transaction-examples)
- [Sign-In Examples](#sign-in-examples)
- [Advanced Usage](#advanced-usage)
- [Error Handling](#error-handling)
- [Real-World Scenarios](#real-world-scenarios)

## Basic Integration

### Simple Wallet Connection

```typescript
import { HyperLinkWalletAdapter } from '@hyperlink/wallet-adapter';
import { WalletAdapterNetwork } from '@solana/wallet-adapter-base';

// Create wallet adapter instance
const wallet = new HyperLinkWalletAdapter({
  clientId: 'your-client-id',
  title: 'My dApp',
  theme: 'system',
  walletAdapterNetwork: WalletAdapterNetwork.Mainnet
});

// Connect to wallet
async function connectWallet() {
  try {
    await wallet.connect();
    console.log('Connected!', wallet.publicKey?.toBase58());
  } catch (error) {
    console.error('Connection failed:', error);
  }
}

// Disconnect from wallet
async function disconnectWallet() {
  try {
    await wallet.disconnect();
    console.log('Disconnected!');
  } catch (error) {
    console.error('Disconnection failed:', error);
  }
}
````

### Wallet Registration

```typescript
import { registerHyperLinkWallet } from "@hyperlink/wallet-adapter";

// Register the wallet
const unregister = registerHyperLinkWallet({
  clientId: "your-client-id",
  title: "My dApp",
  theme: "system",
  rpcUrl: "https://api.mainnet-beta.solana.com",
  installedOnDesktop: true,
  installedOnIos: true,
  installedOnAndroid: false,
  walletAdapterNetwork: WalletAdapterNetwork.Mainnet,
});

// Later, unregister
unregister();
```

## React Integration

### Basic React Component

```typescript
import React, { useState, useEffect } from 'react';
import { HyperLinkWalletAdapter } from '@hyperlink/wallet-adapter';
import { WalletAdapterNetwork } from '@solana/wallet-adapter-base';

function WalletComponent() {
  const [wallet, setWallet] = useState<HyperLinkWalletAdapter | null>(null);
  const [connected, setConnected] = useState(false);
  const [publicKey, setPublicKey] = useState<string | null>(null);

  useEffect(() => {
    const newWallet = new HyperLinkWalletAdapter({
      clientId: 'your-client-id',
      title: 'My dApp',
      theme: 'system',
      walletAdapterNetwork: WalletAdapterNetwork.Mainnet
    });

    // Set up event listeners
    newWallet.on('connect', (pk) => {
      setConnected(true);
      setPublicKey(pk.toBase58());
    });

    newWallet.on('disconnect', () => {
      setConnected(false);
      setPublicKey(null);
    });

    setWallet(newWallet);

    // Cleanup
    return () => {
      newWallet.disconnect();
    };
  }, []);

  const connect = async () => {
    if (wallet) {
      try {
        await wallet.connect();
      } catch (error) {
        console.error('Connection failed:', error);
      }
    }
  };

  const disconnect = async () => {
    if (wallet) {
      try {
        await wallet.disconnect();
      } catch (error) {
        console.error('Disconnection failed:', error);
      }
    }
  };

  return (
    <div>
      {!connected ? (
        <button onClick={connect}>Connect Wallet</button>
      ) : (
        <div>
          <p>Connected: {publicKey}</p>
          <button onClick={disconnect}>Disconnect</button>
        </div>
      )}
    </div>
  );
}

export default WalletComponent;
```

### With React Wallet Adapter

```typescript
import React from 'react';
import { WalletProvider } from '@solana/wallet-adapter-react';
import { WalletModalProvider } from '@solana/wallet-adapter-react-ui';
import { HyperLinkWalletAdapter } from '@hyperlink/wallet-adapter';
import { WalletAdapterNetwork } from '@solana/wallet-adapter-base';

// Import wallet adapter CSS
import '@solana/wallet-adapter-react-ui/styles.css';

const wallets = [
  new HyperLinkWalletAdapter({
    clientId: 'your-client-id',
    title: 'My dApp',
    theme: 'system',
    walletAdapterNetwork: WalletAdapterNetwork.Mainnet
  })
];

function App() {
  return (
    <WalletProvider wallets={wallets} autoConnect>
      <WalletModalProvider>
        <div>
          <h1>My Solana dApp</h1>
          <WalletActions />
        </div>
      </WalletModalProvider>
    </WalletProvider>
  );
}

function WalletActions() {
  const { wallet, connect, disconnect, connected, publicKey } = useWallet();

  const handleConnect = async () => {
    try {
      await connect();
    } catch (error) {
      console.error('Failed to connect:', error);
    }
  };

  const handleDisconnect = async () => {
    try {
      await disconnect();
    } catch (error) {
      console.error('Failed to disconnect:', error);
    }
  };

  return (
    <div>
      {!connected ? (
        <button onClick={handleConnect}>Connect Wallet</button>
      ) : (
        <div>
          <p>Connected: {publicKey?.toBase58()}</p>
          <button onClick={handleDisconnect}>Disconnect</button>
        </div>
      )}
    </div>
  );
}

export default App;
```

## Transaction Examples

### Simple SOL Transfer

```typescript
import { useWallet } from '@solana/wallet-adapter-react';
import { useConnection } from '@solana/wallet-adapter-react';
import { Transaction, SystemProgram, LAMPORTS_PER_SOL, PublicKey } from '@solana/web3.js';

function TransferComponent() {
  const { wallet, publicKey, connected } = useWallet();
  const { connection } = useConnection();
  const [recipient, setRecipient] = useState('');
  const [amount, setAmount] = useState('0.1');
  const [loading, setLoading] = useState(false);

  const sendTransaction = async () => {
    if (!connected || !publicKey || !recipient) return;

    try {
      setLoading(true);

      const transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: publicKey,
          toPubkey: new PublicKey(recipient),
          lamports: parseFloat(amount) * LAMPORTS_PER_SOL
        })
      );

      const signature = await wallet.sendTransaction(transaction, connection);
      console.log('Transaction sent:', signature);

      // Wait for confirmation
      const confirmation = await connection.confirmTransaction(signature);
      if (confirmation.value.err) {
        throw new Error('Transaction failed');
      }

      alert('Transaction successful!');
    } catch (error) {
      console.error('Transaction failed:', error);
      alert('Transaction failed: ' + error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <h3>Send SOL</h3>
      <input
        type="text"
        placeholder="Recipient Public Key"
        value={recipient}
        onChange={(e) => setRecipient(e.target.value)}
      />
      <input
        type="number"
        placeholder="Amount (SOL)"
        value={amount}
        onChange={(e) => setAmount(e.target.value)}
        step="0.01"
        min="0"
      />
      <button
        onClick={sendTransaction}
        disabled={!connected || !recipient || loading}
      >
        {loading ? 'Sending...' : 'Send SOL'}
      </button>
    </div>
  );
}
```

### SPL Token Transfer

```typescript
import { useWallet } from '@solana/wallet-adapter-react';
import { useConnection } from '@solana/wallet-adapter-react';
import { Transaction, PublicKey } from '@solana/web3.js';
import { TOKEN_PROGRAM_ID, createTransferInstruction } from '@solana/spl-token';

function TokenTransferComponent() {
  const { wallet, publicKey, connected } = useWallet();
  const { connection } = useConnection();
  const [recipient, setRecipient] = useState('');
  const [tokenMint, setTokenMint] = useState('');
  const [amount, setAmount] = useState('1');
  const [loading, setLoading] = useState(false);

  const sendToken = async () => {
    if (!connected || !publicKey || !recipient || !tokenMint) return;

    try {
      setLoading(true);

      // Get token accounts
      const fromTokenAccount = await connection.getTokenAccountsByOwner(publicKey, {
        mint: new PublicKey(tokenMint)
      });

      const toTokenAccount = await connection.getTokenAccountsByOwner(
        new PublicKey(recipient),
        { mint: new PublicKey(tokenMint) }
      );

      if (fromTokenAccount.value.length === 0) {
        throw new Error('No token account found for sender');
      }

      if (toTokenAccount.value.length === 0) {
        throw new Error('No token account found for recipient');
      }

      const transaction = new Transaction().add(
        createTransferInstruction(
          fromTokenAccount.value[0].pubkey,
          toTokenAccount.value[0].pubkey,
          publicKey,
          BigInt(parseFloat(amount) * Math.pow(10, 9)) // Assuming 9 decimals
        )
      );

      const signature = await wallet.sendTransaction(transaction, connection);
      console.log('Token transfer sent:', signature);

      const confirmation = await connection.confirmTransaction(signature);
      if (confirmation.value.err) {
        throw new Error('Transaction failed');
      }

      alert('Token transfer successful!');
    } catch (error) {
      console.error('Token transfer failed:', error);
      alert('Token transfer failed: ' + error.message);
    } finally {
      setLoading(false);
    }
  };

  return (
    <div>
      <h3>Send SPL Token</h3>
      <input
        type="text"
        placeholder="Token Mint Address"
        value={tokenMint}
        onChange={(e) => setTokenMint(e.target.value)}
      />
      <input
        type="text"
        placeholder="Recipient Public Key"
        value={recipient}
        onChange={(e) => setRecipient(e.target.value)}
      />
      <input
        type="number"
        placeholder="Amount"
        value={amount}
        onChange={(e) => setAmount(e.target.value)}
        step="0.01"
        min="0"
      />
      <button
        onClick={sendToken}
        disabled={!connected || !recipient || !tokenMint || loading}
      >
        {loading ? 'Sending...' : 'Send Token'}
      </button>
    </div>
  );
}
```

### Multiple Transaction Signing

```typescript
import { useWallet } from '@solana/wallet-adapter-react';
import { useConnection } from '@solana/wallet-adapter-react';
import { Transaction, SystemProgram, LAMPORTS_PER_SOL, PublicKey } from '@solana/web3.js';

function BatchTransferComponent() {
  const { wallet, publicKey, connected } = useWallet();
  const { connection } = useConnection();
  const [recipients, setRecipients] = useState(['', '']);
  const [amount, setAmount] = useState('0.1');
  const [loading, setLoading] = useState(false);

  const sendBatchTransactions = async () => {
    if (!connected || !publicKey || recipients.some(r => !r)) return;

    try {
      setLoading(true);

      const transactions = recipients.map(recipient =>
        new Transaction().add(
          SystemProgram.transfer({
            fromPubkey: publicKey,
            toPubkey: new PublicKey(recipient),
            lamports: parseFloat(amount) * LAMPORTS_PER_SOL
          })
        )
      );

      // Sign all transactions
      const signedTransactions = await wallet.signAllTransactions(transactions);

      // Send each transaction
      const signatures = [];
      for (const signedTx of signedTransactions) {
        const signature = await connection.sendRawTransaction(signedTx.serialize());
        signatures.push(signature);
      }

      console.log('Batch transactions sent:', signatures);
      alert('Batch transactions successful!');
    } catch (error) {
      console.error('Batch transactions failed:', error);
      alert('Batch transactions failed: ' + error.message);
    } finally {
      setLoading(false);
    }
  };

  const addRecipient = () => {
    setRecipients([...recipients, '']);
  };

  const removeRecipient = (index: number) => {
    setRecipients(recipients.filter((_, i) => i !== index));
  };

  const updateRecipient = (index: number, value: string) => {
    const newRecipients = [...recipients];
    newRecipients[index] = value;
    setRecipients(newRecipients);
  };

  return (
    <div>
      <h3>Batch Transfer SOL</h3>
      {recipients.map((recipient, index) => (
        <div key={index}>
          <input
            type="text"
            placeholder={`Recipient ${index + 1} Public Key`}
            value={recipient}
            onChange={(e) => updateRecipient(index, e.target.value)}
          />
          {recipients.length > 1 && (
            <button onClick={() => removeRecipient(index)}>Remove</button>
          )}
        </div>
      ))}
      <button onClick={addRecipient}>Add Recipient</button>
      <input
        type="number"
        placeholder="Amount per recipient (SOL)"
        value={amount}
        onChange={(e) => setAmount(e.target.value)}
        step="0.01"
        min="0"
      />
      <button
        onClick={sendBatchTransactions}
        disabled={!connected || recipients.some(r => !r) || loading}
      >
        {loading ? 'Sending...' : 'Send Batch Transactions'}
      </button>
    </div>
  );
}
```

## Sign-In Examples

### Basic Sign-In

```typescript
import { useWallet } from '@solana/wallet-adapter-react';

function SignInComponent() {
  const { wallet } = useWallet();
  const [loading, setLoading] = useState(false);
  const [signedIn, setSignedIn] = useState(false);

  const handleSignIn = async () => {
    if (!wallet || !('signIn' in wallet)) return;

    try {
      setLoading(true);

      const siwsInput = {
        domain: 'yourdomain.com',
        statement: 'Sign in to access the application',
        uri: 'https://yourdomain.com',
        version: '1',
        chainId: 'solana:mainnet',
        nonce: 'random-nonce-' + Date.now(),
        issuedAt: new Date().toISOString(),
        expirationTime: new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString(),
        notBefore: new Date().toISOString(),
        resources: ['https://yourdomain.com']
      };

      const output = await wallet.signIn(siwsInput);
```
