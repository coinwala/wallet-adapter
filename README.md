# @hyperlink/wallet-adapter

[![npm version](https://badge.fury.io/js/%40hyperlink%2Fwallet-adapter.svg)](https://badge.fury.io/js/%40hyperlink%2Fwallet-adapter)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Node.js >= 18](https://img.shields.io/badge/node-%3E%3D18-brightgreen.svg)](https://nodejs.org/)

A Solana wallet adapter that seamlessly integrates HyperLink wallet functionality into Solana dApps. This library provides a robust, secure, and user-friendly way for users to connect their HyperLink wallet, sign transactions, and interact with Solana blockchain applications.

## ✨ Features

- 🔐 **Solana Wallet Standard Compliance** - Implements the Solana Wallet Standard for broad compatibility
- 📱 **Multi-Platform Support** - Works seamlessly on desktop, iOS, and Android devices
- �� **Sign-In with Solana (SIWS)** - Supports Solana's sign-in standard for authentication
- �� **Transaction Signing** - Sign individual or multiple transactions with ease
- 💬 **Message Signing** - Sign arbitrary messages for authentication and verification
- �� **Auto-Connect** - Automatic wallet reconnection on page reload
- 🎨 **Theme Support** - Light, dark, and system theme options
- 🏦 **Embedded Wallet UI** - Built-in wallet interface with customizable pages
- 🛡️ **Security First** - Allowlist protection and secure communication
- 🚀 **Performance Optimized** - Efficient wallet operations and minimal bundle size

## 📦 Installation

```bash
npm install @hyperlink/wallet-adapter
```

### Peer Dependencies

```json
{
  "@solana/web3.js": "^1.58.0"
}
```

## �� Quick Start

### 1. Register the Wallet Adapter

```typescript
import { registerHyperLinkWallet } from "@hyperlink/wallet-adapter";
import { WalletAdapterNetwork } from "@solana/wallet-adapter-base";

// Register the wallet adapter
const unregister = registerHyperLinkWallet({
  clientId: "YOUR_CLIENT_ID", // Get this from HyperLink team
  title: "My dApp",
  theme: "system", // 'light', 'dark', or 'system'
  rpcUrl: "https://api.mainnet-beta.solana.com",
  installedOnDesktop: true,
  installedOnIos: true,
  installedOnAndroid: false,
  walletAdapterNetwork: WalletAdapterNetwork.Mainnet,
});

// Clean up when done
unregister();
```

### 2. Use with React Wallet Adapter

```typescript
import { WalletProvider } from '@solana/wallet-adapter-react';
import { HyperLinkWalletAdapter } from '@hyperlink/wallet-adapter';
import { WalletAdapterNetwork } from '@solana/wallet-adapter-base';

const wallets = [
  new HyperLinkWalletAdapter({
    clientId: 'YOUR_CLIENT_ID',
    title: 'My dApp',
    theme: 'system',
    installedOnDesktop: true,
    installedOnIos: true,
    installedOnAndroid: false,
    walletAdapterNetwork: WalletAdapterNetwork.Mainnet
  })
];

function App() {
  return (
    <WalletProvider wallets={wallets} autoConnect>
      {/* Your app components */}
    </WalletProvider>
  );
}
```

### 3. Connect and Use the Wallet

```typescript
import { useWallet } from '@solana/wallet-adapter-react';
import { useConnection } from '@solana/wallet-adapter-react';

function WalletActions() {
  const { wallet, connect, disconnect, connected, publicKey } = useWallet();
  const { connection } = useConnection();

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
```

## �� Documentation

- [API Reference](./docs/API.md)
- [Configuration Guide](./docs/CONFIGURATION.md)
- [Examples](./docs/EXAMPLES.md)
- [Troubleshooting](./docs/TROUBLESHOOTING.md)
- [Migration Guide](./docs/MIGRATION.md)

## �� Configuration

### WalletAdapterConfig

```typescript
interface WalletAdapterConfig {
  clientId: string; // Required: Get from HyperLink team
  title: string; // Required: Your dApp name
  theme: "system" | "light" | "dark"; // Required: UI theme preference
  installedOnDesktop?: boolean; // Optional: Desktop support (default: true)
  installedOnIos?: boolean; // Optional: iOS support (default: true)
  installedOnAndroid?: boolean; // Optional: Android support (default: false)
  hideDraggableWidget?: boolean; // Optional: Hide draggable widget (default: false)
  hideWalletOnboard?: boolean; // Optional: Hide wallet onboarding (default: false)
  walletAdapterNetwork?:
    | WalletAdapterNetwork.Mainnet
    | WalletAdapterNetwork.Devnet;
}
```

## 🌟 Advanced Features

### Sign-In with Solana (SIWS)

```typescript
import { useWallet } from '@solana/wallet-adapter-react';

function SignInComponent() {
  const { wallet } = useWallet();

  const handleSignIn = async () => {
    if (wallet && 'signIn' in wallet) {
      try {
        const siwsInput = {
          domain: 'yourdomain.com',
          statement: 'Sign in to access the application',
          uri: 'https://yourdomain.com',
          version: '1',
          chainId: 'solana:mainnet',
          nonce: 'random-nonce',
          issuedAt: new Date().toISOString(),
          expirationTime: new Date(Date.now() + 24 * 60 * 60 * 1000).toISOString(),
          notBefore: new Date().toISOString(),
          resources: ['https://yourdomain.com']
        };

        const output = await wallet.signIn(siwsInput);
        console.log('Sign-in successful:', output);
      } catch (error) {
        console.error('Sign-in failed:', error);
      }
    }
  };

  return <button onClick={handleSignIn}>Sign In with Solana</button>;
}
```

### Transaction Signing

```typescript
import { useWallet } from '@solana/wallet-adapter-react';
import { Transaction, SystemProgram, LAMPORTS_PER_SOL, PublicKey } from '@solana/web3.js';

function TransactionComponent() {
  const { wallet, publicKey, connected } = useWallet();

  const sendTransaction = async () => {
    if (!connected || !publicKey) return;

    try {
      const transaction = new Transaction().add(
        SystemProgram.transfer({
          fromPubkey: publicKey,
          toPubkey: new PublicKey('RECIPIENT_PUBLIC_KEY'),
          lamports: LAMPORTS_PER_SOL * 0.1
        })
      );

      const signature = await wallet.sendTransaction(transaction, connection);
      console.log('Transaction sent:', signature);
    } catch (error) {
      console.error('Transaction failed:', error);
    }
  };

  return <button onClick={sendTransaction}>Send Transaction</button>;
}
```

### Message Signing

```typescript
import { useWallet } from '@solana/wallet-adapter-react';

function MessageSigningComponent() {
  const { wallet, connected } = useWallet();

  const signMessage = async () => {
    if (!connected) return;

    try {
      const message = new TextEncoder().encode('Hello, Solana!');
      const signature = await wallet.signMessage(message);
      console.log('Message signed:', signature);
    } catch (error) {
      console.error('Message signing failed:', error);
    }
  };

  return <button onClick={signMessage}>Sign Message</button>;
}
```

### Embedded Wallet Pages

```typescript
import { useWallet } from '@solana/wallet-adapter-react';
import { EmbeddedWalletPage } from '@hyperlink/wallet-adapter';

function WalletControls() {
  const { wallet } = useWallet();

  const showWalletPage = (page?: EmbeddedWalletPage) => {
    if (wallet && 'showWallet' in wallet) {
      wallet.showWallet(page);
    }
  };

  const hideWallet = () => {
    if (wallet && 'hideWallet' in wallet) {
      wallet.hideWallet();
    }
  };

  return (
    <div>
      <button onClick={() => showWalletPage(EmbeddedWalletPage.OVERVIEW)}>
        Show Overview
      </button>
      <button onClick={() => showWalletPage(EmbeddedWalletPage.ADD_FUNDS)}>
        Add Funds
      </button>
      <button onClick={() => showWalletPage(EmbeddedWalletPage.SWAP)}>
        Swap
      </button>
      <button onClick={() => showWalletPage(EmbeddedWalletPage.WITHDRAW)}>
        Withdraw
      </button>
      <button onClick={hideWallet}>Hide Wallet</button>
    </div>
  );
}
```

## 🌐 Browser Compatibility

The wallet adapter automatically detects the user's environment and adjusts behavior accordingly:

- **Desktop Browsers**: Full functionality with direct connection
- **Mobile Browsers**: Optimized for mobile with iframe fallback
- **PWA Mode**: Automatic iframe mode for progressive web apps
- **In-App Browsers**: Graceful degradation with user guidance

## ��️ Security Features

- **Allowlist Protection**: Client ID and domain validation
- **Secure Communication**: Encrypted wallet communication
- **Session Management**: Secure session handling with UUID generation
- **Origin Validation**: Referrer URL validation for security

## 🚀 Development

### Prerequisites

- Node.js >= 18
- npm >= 9.1.0

### Build Commands

```bash
# Install dependencies
npm install

# Build the library
npm run build

# Build for release
npm run build:release

# Lint and format
npm run lint

# Clean build artifacts
npm run clean

# Create package for publishing
npm run pack
```

### Project Structure
