# Upland-portfolio
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import "@openzeppelin/contracts/token/ERC721/ERC721.sol";
import "@openzeppelin/contracts/access/Ownable.sol";
import "@openzeppelin/contracts/utils/Counters.sol";

/// @title Upland Land Registry
/// @notice ERC721-based land registry for minting and tracking land parcels.
contract LandRegistry is ERC721, Ownable {
    using Counters for Counters.Counter;

    struct Parcel {
        string name;
        uint256 x;
        uint256 y;
        uint256 size;
        string metadataURI;
    }

    Counters.Counter private _tokenIds;
    mapping(uint256 => Parcel) private _parcels;
    mapping(bytes32 => bool) private _parcelCoordinates;

    event LandRegistered(
        uint256 indexed tokenId,
        address indexed owner,
        string name,
        uint256 x,
        uint256 y,
        uint256 size,
        string metadataURI
    );

    event ParcelMetadataUpdated(uint256 indexed tokenId, string metadataURI);

    constructor() ERC721("UplandLand", "UPLAND") {}

    /// @notice Register a new parcel and mint its ERC721 token.
    /// @dev Only the contract owner can register new parcels.
    /// @param to Recipient of the new land token.
    /// @param name Friendly name for the parcel.
    /// @param x X coordinate of the parcel.
    /// @param y Y coordinate of the parcel.
    /// @param size Parcel size in square meters.
    /// @param metadataURI Optional token metadata URI.
    /// @return tokenId The minted token ID associated with the parcel.
    function registerLand(
        address to,
        string calldata name,
        uint256 x,
        uint256 y,
        uint256 size,
        string calldata metadataURI
    ) external onlyOwner returns (uint256) {
        require(to != address(0), "LandRegistry: invalid recipient");
        require(size > 0, "LandRegistry: size must be positive");

        bytes32 coordinateHash = keccak256(abi.encodePacked(x, y));
        require(!_parcelCoordinates[coordinateHash], "LandRegistry: parcel already exists");

        _tokenIds.increment();
        uint256 tokenId = _tokenIds.current();

        _parcels[tokenId] = Parcel({
            name: name,
            x: x,
            y: y,
            size: size,
            metadataURI: metadataURI
        });
        _parcelCoordinates[coordinateHash] = true;

        _safeMint(to, tokenId);
        emit LandRegistered(tokenId, to, name, x, y, size, metadataURI);

        return tokenId;
    }

    /// @notice Update the metadata URI for an existing parcel.
    /// @dev Only the contract owner may update metadata.
    /// @param tokenId The token ID of the parcel.
    /// @param metadataURI New metadata URI.
    function updateParcelMetadata(uint256 tokenId, string calldata metadataURI) external onlyOwner {
        require(_exists(tokenId), "LandRegistry: token does not exist");
        _parcels[tokenId].metadataURI = metadataURI;
        emit ParcelMetadataUpdated(tokenId, metadataURI);
    }

    /// @notice Returns parcel details for a given token ID.
    /// @param tokenId The token ID to query.
    function parcelDetails(uint256 tokenId)
        external
        view
        returns (
            string memory name,
            uint256 x,
            uint256 y,
            uint256 size,
            string memory metadataURI
        )
    {
        require(_exists(tokenId), "LandRegistry: token does not exist");
        Parcel storage parcel = _parcels[tokenId];
        return (parcel.name, parcel.x, parcel.y, parcel.size, parcel.metadataURI);
    }

    /// @inheritdoc ERC721
    function tokenURI(uint256 tokenId) public view override returns (string memory) {
        require(_exists(tokenId), "LandRegistry: URI query for nonexistent token");
        return _parcels[tokenId].metadataURI;
    }
}
