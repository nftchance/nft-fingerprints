# Fingerprints

Fingerprint is an extension that enables the token-level gating of access without the concern of gas costs or rampant token re-use. Traditionally, much of the Ethereum  ecosystem has relied on the use of a single token to represent access to a particular while then resolving that credential to the user, instead of the token: which was the actual key, the user merely turned the key in the lock.

Fingerprint solves this problem by creating a unique fingerprint for each token, which is then used to gate access to the resource. This fingerprint is a 32 byte hash, which is generated by hashing the token's ID and the user's address. This fingerprint is then used to gate access to the resource, and is not re-usable until the fingerprint decay  reached zero. The stored fingerprint is also not accessible on-chain, and is only generated when the user  attempts to gain access to the resource. This means, that the token can be sold, traded, or stolen, and the user will be able to re-use the key once the fingerprint has decayed to zero. Allowing for normal market activites to occur without the concern of the token being re-used before you deem acceptable.

- **Important:** Only works for ERC721 tokens.

## Use Cases

- Implement token-level access control without losing ability to have traditional market activities.
- Implement token-level access control without the concern of gas costs.
- Implement token-level access control without the concern of rampant token re-use.
- Implement token-level access control without the concern of the token being re-used before you deem acceptable.

## Usage

To confirm that a user has access to a resource you can use the `onlyVirginFingerprints` modifier on any gated function:

```solidity
    /**
     * @notice Enforces that any fingerprint being used to gain access to the resource has decayed to zero
     *         or has not yet been used.
     * @param _fingerprint The fingerprint to check the decay of.
     *
     * Requirements:
     * - The fingerprint must have decayed to zero or not yet been used.
     */
    modifier onlyVirginFingerprints(
        bytes32 _fingerprint
    ) { 
        int256 virginState = fingerprints[_fingerprint] - fingerprintDecayRate;

        require(
              virginState < 1
            , "Fingerprint: Fingerprint already exists");
        _;
    }
```

To set the fingerprint for a token, you can use the `_setFingerprint` function:

```solidity
    /**
     * @notice Sets the fingerprint for a token.
     * @param _tokenId The ID of the token to set the fingerprint for.
     * @param _fingerprint The fingerprint to set for the token.
     *
     * Requirements:
     * - The fingerprint must not already exist.
     */
    function _setFingerprint(
        uint256 _tokenId,
        bytes32 _fingerprint
    ) 
        internal 
        onlyVirginFingerprints(_fingerprint) 
    {
        fingerprints[_fingerprint] = fingerprintDecayRate;
    }
```