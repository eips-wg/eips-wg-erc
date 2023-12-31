---
eip: 7007
title: Zero-Knowledge AI-Generated Content Token
description: An ERC-721 extension interface for zkML based AIGC-NFTs.
author: Cathie So (@socathie), Xiaohang Yu (@xhyumiracle), Huaizhe Xu (@HuaizheXu), Kartin <kartin@hyperoracle.io>
discussions-to: https://ethereum-magicians.org/t/eip-7007-zkml-aigc-nfts-an-erc-721-extension-interface-for-zkml-based-aigc-nfts/14216
status: Draft
type: Standards Track
category: ERC
created: 2023-05-10
requires: 165, 721
---

## Abstract

The Zero-Knowledge Machine Lerning (zkML) AI-Generated Content (AIGC) non-fungible token (NFT) standard is an extension of the [ERC-721](https://eips.fyi/721) token standard for AIGC. It proposes a set of interfaces for basic interactions and enumerable interactions for AIGC-NFTs. The standard includes a new mint event and a JSON schema for AIGC-NFT metadata. Additionally, it incorporates zkML capabilities to enable verification of AIGC-NFT ownership. In this standard, the `tokenId` is indexed by the `prompt`.

## Motivation

The zkML AIGC-NFTs standard aims to extend the existing [ERC-721](https://eips.fyi/721) token standard to accommodate the unique requirements of AI-Generated Content NFTs representing models in a collection. This standard provides interfaces to use zkML to verify whether or not the AIGC data for an NFT is generated from a certain ML model with certain input (prompt). The proposed interfaces allow for additional functionality related to minting, verifying, and enumerating AIGC-NFTs. Additionally, the metadata schema provides a structured format for storing information related to AIGC-NFTs, such as the prompt used to generate the content and the proof of ownership.

With this standard, model owners can publish their trained model and its ZKP verifier to Ethereum. Any user can claim an input (prompt) and publish the inference task, any node that maintains the model and the proving circuit can perform the inference and proving, then submit the output of inference and the ZK proof for the inference trace into the verifier that is deployed by the model owner. The user that initiates the inference task will own the output for the inference of that model and input (prompt).

## Specification

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

**Every compliant contract must implement the [ERC-7007](https://eips.fyi/7007), [ERC-721](https://eips.fyi/721), and [ERC-165](https://eips.fyi/165) interfaces.**

The zkML AIGC-NFTs standard includes the following interfaces: 

`IERC7007`: Defines a mint event and a mint function for minting AIGC-NFTs. It also includes a verify function to check the validity of a combination of prompt and proof using zkML techniques.

```solidity
pragma solidity ^0.8.18;

/**
 * @dev Required interface of an ERC7007 compliant contract.
 * Note: the ERC-165 identifier for this interface is 0x7e52e423.
 */
interface IERC7007 is IERC165, IERC721 {
    /**
     * @dev Emitted when `tokenId` token is minted.
     */
    event Mint(
        uint256 indexed tokenId,
        bytes indexed prompt,
        bytes indexed aigcData,
        string uri,
        bytes proof
    );

    /**
     * @dev Mint token at `tokenId` given `prompt`, `aigcData`, `uri` and `proof`.
     *
     * Requirements:
     * - `tokenId` must not exist.'
     * - verify(`prompt`, `aigcData`, `proof`) must return true.
     *
     * Optional:
     * - `proof` should not include `aigcData` to save gas.
     */
    function mint(
        bytes calldata prompt,
        bytes calldata aigcData,
        string calldata uri,
        bytes calldata proof
    ) external returns (uint256 tokenId);

    /**
     * @dev Verify the `prompt`, `aigcData` and `proof`.
     */
    function verify(
        bytes calldata prompt,
        bytes calldata aigcData,
        bytes calldata proof
    ) external view returns (bool success);
}
```

Optional Extension: Enumerable

The **enumeration extension** is OPTIONAL for [ERC-7007](https://eips.fyi/7007) smart contracts. This allows your contract to publish its full list of mapping between `tokenId` and `prompt` and make them discoverable.

```solidity
pragma solidity ^0.8.18;

/**
 * @title ERC7007 Token Standard, optional enumeration extension
 * Note: the ERC-165 identifier for this interface is 0xfa1a557a.
 */
interface IERC7007Enumerable is IERC7007 {
    /**
     * @dev Returns the token ID given `prompt`.
     */
    function tokenId(bytes calldata prompt) external view returns (uint256);

    /**
     * @dev Returns the prompt given `tokenId`.
     */
    function prompt(uint256 tokenId) external view returns (string calldata);
}
```

ERC-7007 Metadata JSON Schema for reference

```json
{
    "title": "AIGC Metadata",
    "type": "object",
    "properties": {
        "name": {
            "type": "string",
            "description": "Identifies the asset to which this NFT represents"
        },
        "description": {
            "type": "string",
            "description": "Describes the asset to which this NFT represents"
        },
        "image": {
            "type": "string",
            "description": "A URI pointing to a resource with mime type image/* representing the asset to which this NFT represents. Consider making any images at a width between 320 and 1080 pixels and aspect ratio between 1.91:1 and 4:5 inclusive."
        },

        "prompt": {
            "type": "string",
            "description": "Identifies the prompt from which this AIGC NFT generated"
        },
        "aigc_type": {
            "type": "string",
            "description": "image/video/audio..."
        },
        "aigc_data": {
            "type": "string",
            "description": "A URI pointing to a resource with mime type image/* representing the asset to which this AIGC NFT represents."
        }
    }
}
```

## Rationale

TBD

## Backwards Compatibility

This standard is backward compatible with the [ERC-721](https://eips.fyi/721) as it extends the existing functionality with new interfaces.

## Test Cases

The reference implementation includes sample implementations of the [ERC-7007](https://eips.fyi/7007) interfaces under `contracts/` and corresponding unit tests under `test/`. This repo can be used to test the functionality of the proposed interfaces and metadata schema.

## Reference Implementation

* [ERC-7007](./assets/contracts/ERC7007.sol)
* [ERC-7007 Enumerable Extension](./assets/contracts/ERC7007Enumerable.sol)

## Security Considerations

Needs discussion.

## Copyright

Copyright and related rights waived via [CC0](/LICENSE.md).
