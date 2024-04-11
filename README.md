- 👋 Hi, I’m @tietougongtt
- 👀 I’m interested in ...
- 🌱 I’m currently learning ...
- 💞️ I’m looking to collaborate on ...
- 📫 How to reach me ...
- 😄 Pronouns: ...
- ⚡ Fun fact: ...

<!---
tietougongtt/tietougongtt is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->
pragma solidity ^0.8.0;

import "./IPFSStorage.sol";

contract ArtMarketplace {
    IPFSStorage private ipfsStorage;

    struct Artwork {
        address owner;
        string ipfsHash;
        string story;
    }

    mapping(uint256 => Artwork) public artworks;

    event ArtworkUploaded(uint256 indexed tokenId, address owner);
    event ArtworkTransferred(uint256 indexed tokenId, address from, address to);

    constructor(address _ipfsStorageAddress) {
        ipfsStorage = IPFSStorage(_ipfsStorageAddress);
    }

    function uploadArtwork(string memory ipfsHash, string memory story) public {
        uint256 tokenId = ipfsStorage.upload(ipfsHash);
        
        Artwork memory newArtwork = Artwork(msg.sender, ipfsHash, story);
        artworks[tokenId] = newArtwork;

        emit ArtworkUploaded(tokenId, msg.sender);
    }

    function transferArtwork(uint256 tokenId, address to, string memory newStory) public {
        Artwork storage art = artworks[tokenId];
        address owner = art.owner;
        require(owner == msg.sender, "You are not the owner of this artwork");
        
        art.owner = to;
        art.story = newStory;

        emit ArtworkTransferred(tokenId, msg.sender, to);
    }

    function buyArtwork(uint256 tokenId, string memory newStory) public payable {
        Artwork storage art = artworks[tokenId];
        address owner = art.owner;
        require(owner != address(0), "Artwork does not exist");
        require(owner != msg.sender, "You already own this artwork");
        
        address payable ownerPayable = payable(owner);
        ownerPayable.transfer(msg.value);
        
        art.owner = msg.sender;
        art.story = newStory;

        emit ArtworkTransferred(tokenId, owner, msg.sender);
    }

    function getArtworkDetails(uint256 tokenId) public view returns (address owner, string memory ipfsHash, string memory story) {
        Artwork memory art = artworks[tokenId];
        return (art.owner, art.ipfsHash, art.story);
    }
}