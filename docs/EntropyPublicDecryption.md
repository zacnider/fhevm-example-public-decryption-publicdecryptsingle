# EntropyPublicDecryption

Learn how to public decrypt single value using encrypted randomness and makepubliclydecryptable

## 📚 Overview

@title EntropyPublicDecryption
@notice Public decrypt single value using encrypted randomness and makePubliclyDecryptable
@dev This example teaches you how to integrate encrypted randomness into your FHEVM contracts: using entropy for public decryption patterns
In this example, you will learn:
- How to integrate encrypted randomness
- How to use encrypted randomness to enhance public decryption patterns
- Combining entropy with public decryption
- Entropy-based public key generation

@notice Constructor - sets encrypted randomness address
@param _encrypted randomness Address of encrypted randomness contract

@notice Store encrypted value and make it publicly decryptable
@param encryptedInput Encrypted value from user
@param inputProof Input proof for encrypted value
@dev Makes value decryptable by anyone (use with caution)

@notice Request entropy for enhanced public decryption
@param tag Unique tag for this request
@return requestId Request ID from encrypted randomness
@dev Requires 0.00001 ETH fee

@notice Store value with entropy enhancement and make publicly decryptable
@param encryptedInput Encrypted value from user
@param inputProof Input proof for encrypted value
@param requestId Request ID from requestEntropy()

@notice Get encrypted value (publicly decryptable)
@return Encrypted value (euint64) - anyone can decrypt this
@dev Anyone can use FHEVM SDK publicDecrypt to decrypt this value

@notice Check if initialized

@notice Get encrypted randomness address



## Contract Code

```solidity
// SPDX-License-Identifier: BSD-3-Clause-Clear
pragma solidity ^0.8.27;

import {FHE, euint64, externalEuint64} from "@fhevm/solidity/lib/FHE.sol";
import {ZamaEthereumConfig} from "@fhevm/solidity/config/ZamaConfig.sol";
import "./IEntropyOracle.sol";

/**
 * @title EntropyPublicDecryption
 * @notice Public decrypt single value using EntropyOracle and makePubliclyDecryptable
 * @dev Example demonstrating EntropyOracle integration: using entropy for public decryption patterns
 * 
 * This example shows:
 * - How to integrate with EntropyOracle
 * - Using entropy to enhance public decryption patterns
 * - Combining entropy with public decryption
 * - Entropy-based public key generation
 */
contract EntropyPublicDecryption is ZamaEthereumConfig {
    // Entropy Oracle interface
    IEntropyOracle public entropyOracle;
    
    // Encrypted value (publicly decryptable)
    euint64 private encryptedValue;
    
    bool private initialized;
    
    // Track entropy requests
    mapping(uint256 => bool) public entropyRequests;
    
    event ValueStored(address indexed user);
    event ValueMadePubliclyDecryptable();
    event EntropyRequested(uint256 indexed requestId, address indexed caller);
    event ValueStoredWithEntropy(uint256 indexed requestId, address indexed user);
    
    /**
     * @notice Constructor - sets EntropyOracle address
     * @param _entropyOracle Address of EntropyOracle contract
     */
    constructor(address _entropyOracle) {
        require(_entropyOracle != address(0), "Invalid oracle address");
        entropyOracle = IEntropyOracle(_entropyOracle);
    }
    
    /**
     * @notice Store encrypted value and make it publicly decryptable
     * @param encryptedInput Encrypted value from user
     * @param inputProof Input proof for encrypted value
     * @dev Makes value decryptable by anyone (use with caution)
     */
    function storeAndMakePublic(
        externalEuint64 encryptedInput,
        bytes calldata inputProof
    ) external {
        require(!initialized, "Already initialized");
        
        // Convert external to internal
        euint64 internalValue = FHE.fromExternal(encryptedInput, inputProof);
        
        // Allow contract to use
        FHE.allowThis(internalValue);
        
        // Make publicly decryptable (anyone can decrypt)
        encryptedValue = FHE.makePubliclyDecryptable(internalValue);
        initialized = true;
        
        emit ValueStored(msg.sender);
        emit ValueMadePubliclyDecryptable();
    }
    
    /**
     * @notice Request entropy for enhanced public decryption
     * @param tag Unique tag for this request
     * @return requestId Request ID from EntropyOracle
     * @dev Requires 0.00001 ETH fee
     */
    function requestEntropy(bytes32 tag) external payable returns (uint256 requestId) {
        require(msg.value >= entropyOracle.getFee(), "Insufficient fee");
        
        requestId = entropyOracle.requestEntropy{value: msg.value}(tag);
        entropyRequests[requestId] = true;
        
        emit EntropyRequested(requestId, msg.sender);
        return requestId;
    }
    
    /**
     * @notice Store value with entropy enhancement and make publicly decryptable
     * @param encryptedInput Encrypted value from user
     * @param inputProof Input proof for encrypted value
     * @param requestId Request ID from requestEntropy()
     */
    function storeAndMakePublicWithEntropy(
        externalEuint64 encryptedInput,
        bytes calldata inputProof,
        uint256 requestId
    ) external {
        require(!initialized, "Already initialized");
        require(entropyRequests[requestId], "Invalid request ID");
        require(entropyOracle.isRequestFulfilled(requestId), "Entropy not ready");
        
        // Convert external to internal
        euint64 internalValue = FHE.fromExternal(encryptedInput, inputProof);
        FHE.allowThis(internalValue);
        
        // Get entropy
        euint64 entropy = entropyOracle.getEncryptedEntropy(requestId);
        FHE.allowThis(entropy);
        
        // Combine value with entropy
        euint64 enhancedValue = FHE.xor(internalValue, entropy);
        FHE.allowThis(enhancedValue);
        
        // Make enhanced value publicly decryptable
        encryptedValue = FHE.makePubliclyDecryptable(enhancedValue);
        initialized = true;
        
        entropyRequests[requestId] = false;
        emit ValueStoredWithEntropy(requestId, msg.sender);
        emit ValueMadePubliclyDecryptable();
    }
    
    /**
     * @notice Get encrypted value (publicly decryptable)
     * @return Encrypted value (euint64) - anyone can decrypt this
     * @dev Anyone can use FHEVM SDK publicDecrypt to decrypt this value
     */
    function getEncryptedValue() external view returns (euint64) {
        require(initialized, "Not initialized");
        return encryptedValue;
    }
    
    /**
     * @notice Check if initialized
     */
    function isInitialized() external view returns (bool) {
        return initialized;
    }
    
    /**
     * @notice Get EntropyOracle address
     */
    function getEntropyOracle() external view returns (address) {
        return address(entropyOracle);
    }
}

```

## Tests

See [test file](../examples/public-decryption-publicdecryptsingle/test/EntropyPublicDecryption.test.ts) for comprehensive test coverage.

```bash
npm test
```


## Category

**public**



## Related Examples

- [All public examples](../examples/public/)
