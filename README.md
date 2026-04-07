# Upland-portfolio
# Upland Portfolio

## Overview

`Upland Portfolio` is a small Solidity project containing an ERC721 land registry smart contract for minting and tracking virtual land parcels.

The registry contract is implemented in `contracts/LandRegistry.sol` and includes:

- ERC721 tokenization of land parcels
- owner-only land registration
- unique coordinate enforcement for each parcel
- parcel metadata storage and retrieval
- token URI support via on-chain metadata URI

## Contract

- `contracts/LandRegistry.sol`

### Key functions

- `registerLand(address to, string name, uint256 x, uint256 y, uint256 size, string metadataURI) external onlyOwner returns (uint256)`
  - Mints a new parcel token and assigns it to `to`
  - Prevents duplicate parcels at the same `x, y` coordinate

- `updateParcelMetadata(uint256 tokenId, string metadataURI) external onlyOwner`
  - Updates the metadata URI for an existing parcel

- `parcelDetails(uint256 tokenId) external view returns (string memory name, uint256 x, uint256 y, uint256 size, string memory metadataURI)`
  - Returns stored parcel details for a token

- `tokenURI(uint256 tokenId) public view override returns (string memory)`
  - Returns the token metadata URI for ERC721 compatibility

## Development

This repository currently includes the contract source only. To compile, test, or deploy, create a Solidity development project using a framework such as Hardhat, Foundry, or Truffle.

Example using Hardhat:

```bash
npm init -y
npm install --save-dev hardhat @nomicfoundation/hardhat-toolbox @openzeppelin/contracts
npx hardhat
```

Then move `contracts/LandRegistry.sol` into the Hardhat project `contracts/` folder and use `npx hardhat compile`.

## Notes

- The contract is written for Solidity `^0.8.20`.
- `Ownable` restricts land registration and metadata updates to the contract owner.
- Coordinate uniqueness is enforced with `keccak256(abi.encodePacked(x, y))`.

## Next steps

- Add tests for parcel registration and metadata updates
- Add deployment scripts
- Integrate a UI or script that interacts with the contract

