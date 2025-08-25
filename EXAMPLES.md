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
import { HyperLinkWalletAdapter } from "@hyperlink/wallet-adapter";
import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";

// Create wallet adapter instance
const wallet = new HyperLinkWalletAdapter({
  clientId: "your-client-id",
  title: "My dApp",
  theme: "system",
  walletAdapterNetwork: WalletAdapterNetwork.Mainnet,
});

// Connect to wallet
async function connectWallet() {
  try {
    await wallet.connect();
    console.log("Connected!", wallet.publicKey?.toBase58());
  } catch (error) {
    console.error("Connection failed:", error);
  }
}

// Disconnect from wallet
async function disconnectWallet() {
  try {
    await wallet.disconnect();
    console.log("Disconnected!");
  } catch (error) {
    console.error("Disconnection failed:", error);
  }
}
```

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
