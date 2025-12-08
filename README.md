# EntropyPublicDecryption

Public decrypt single value using EntropyOracle and makePubliclyDecryptable

## 🚀 Standard workflow
- Install (first run): `npm install --legacy-peer-deps`
- Compile: `npx hardhat compile`
- Test (local FHE + local oracle/chaos engine auto-deployed): `npx hardhat test`
- Deploy (frontend Deploy button): constructor arg is fixed to EntropyOracle `0x75b923d7940E1BD6689EbFdbBDCD74C1f6695361`
- Verify: `npx hardhat verify --network sepolia <contractAddress> 0x75b923d7940E1BD6689EbFdbBDCD74C1f6695361`

## 📋 Overview

This example demonstrates **public-decryption** concepts in FHEVM with **EntropyOracle integration**:
- Integrating with EntropyOracle
- Using entropy to enhance public decryption patterns
- Combining entropy with public decryption
- Entropy-based public key generation

## 💡 Key Concepts

### EntropyOracle Integration
The contract uses EntropyOracle to get encrypted randomness for enhanced public decryption:
```solidity
IEntropyOracle entropyOracle;
uint256 requestId = entropyOracle.requestEntropy{value: fee}(tag);
euint64 entropy = entropyOracle.getEncryptedEntropy(requestId);
```

### Entropy-Enhanced Public Decryption
Instead of simple public decryption, values can be enhanced with entropy:
- User encrypts value off-chain
- Request entropy from EntropyOracle
- Combine user value with entropy using XOR
- Make enhanced value publicly decryptable
- Result: Entropy-enhanced public decryption

### Basic Public Decryption
Standard public decryption is still available:
- `storeAndMakePublic()` - Store value and make it publicly decryptable

### FHE.makePubliclyDecryptable Pattern
Uses `FHE.makePubliclyDecryptable()` to allow anyone to decrypt:
```solidity
encryptedValue = FHE.makePubliclyDecryptable(internalValue);
```

## 🚀 Quick Start

### Prerequisites

- Node.js 18+
- Hardhat
- Sepolia Testnet (for FHEVM)
- **Deployed EntropyOracle contract** (required for entropy-based decryption)

### Installation

```bash
npm install
```

### Compile

```bash
npm run compile
```

### Test

```bash
npm test
```

**Note**: Full entropy tests require a deployed EntropyOracle contract. Set `ENTROPY_ORACLE_ADDRESS` environment variable.

## 📖 Usage Example

### Basic Usage (Without Entropy)

```typescript
// Deploy contract
const contract = await EntropyPublicDecryptionFactory.deploy(oracleAddress);

// Store and make public
const input = hre.fhevm.createEncryptedInput(contractAddress, userAddress);
input.add64(42);
const encryptedInput = await input.encrypt();

await contract.storeAndMakePublic(
  encryptedInput.handles[0],
  encryptedInput.inputProof
);
```

### Entropy-Enhanced Usage

```typescript
// Request entropy
const tag = ethers.id("public-decrypt-1");
const fee = await contract.entropyOracle.getFee();
const requestId = await contract.requestEntropy(tag, { value: fee });

// Wait for entropy to be ready
await waitForEntropy(requestId);

// Store with entropy
const input = hre.fhevm.createEncryptedInput(contractAddress, userAddress);
input.add64(42);
const encryptedInput = await input.encrypt();

await contract.storeAndMakePublicWithEntropy(
  encryptedInput.handles[0],
  encryptedInput.inputProof,
  requestId
);
```

## 🔗 Related Examples

- [EntropyUserDecryption](../user-decryption-userdecryptsingle/) - Entropy-based user decryption
- [EntropyAccessControl](../access-control-accesscontrol/) - Entropy-based access control
- [Category: public-decryption](../)

## 📝 License

BSD-3-Clause-Clear
