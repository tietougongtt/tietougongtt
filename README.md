- 👋 Hi, I’m @tietougongtt

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

构思2：为了增加创作形式的多样性，并让创作者的作品成为可编程IP，我们可以结合智能合约和NFT技术，使创作者能够将各种形式的创作内容转化为NFT，并添加一些保护措施来确保创作者的利益。以下是一个示例合约，结合了多样化的创作形式和保护措施：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC721/extensions/ERC721Enumerable.sol";
import "@openzeppelin/contracts/access/Ownable.sol";

contract MyContentMarketplace is ERC721Enumerable, Ownable {
    uint256 private _tokenIdCounter;

    // 存储NFT的创作内容
    mapping(uint256 => string) public tokenContent;

    // 存储NFT的创作者地址
    mapping(uint256 => address) public tokenCreators;

    // 存储NFT的销售价格
    mapping(uint256 => uint256) public tokenPrices;

    // 存储NFT的授权地址
    mapping(uint256 => address) public tokenApprovals;

    // 事件用于记录创作内容上传和销售
    event ContentUploaded(uint256 indexed tokenId, string content, address indexed creator);
    event TokenListed(uint256 indexed tokenId, uint256 price);
    event TokenSold(uint256 indexed tokenId, address indexed buyer, uint256 price);

    constructor() ERC721("MyNFT", "MNFT") {}

    // 上传创作内容，并生成对应的NFT
    function uploadContent(string memory content) public returns (uint256) {
        require(bytes(content).length > 0, "Content cannot be empty");
        _tokenIdCounter++;
        uint256 newTokenId = _tokenIdCounter;
        _mint(msg.sender, newTokenId);
        tokenContent[newTokenId] = content;
        tokenCreators[newTokenId] = msg.sender;
        emit ContentUploaded(newTokenId, content, msg.sender);
        return newTokenId;
    }

    // 获取NFT的创作内容
    function getContent(uint256 tokenId) public view returns (string memory) {
        return tokenContent[tokenId];
    }

    // 设置NFT的销售价格
    function setPrice(uint256 tokenId, uint256 price) public {
        require(_exists(tokenId), "Token does not exist");
        require(ownerOf(tokenId) == msg.sender, "Only token owner can set price");
        tokenPrices[tokenId] = price;
        emit TokenListed(tokenId, price);
    }

    // 购买NFT
    function buyToken(uint256 tokenId) public payable {
        require(_exists(tokenId), "Token does not exist");
        require(tokenPrices[tokenId] > 0, "Token is not listed for sale");
        require(msg.value >= tokenPrices[tokenId], "Insufficient funds");
        address tokenOwner = ownerOf(tokenId);
        tokenApprovals[tokenId] = msg.sender;
        safeTransferFrom(tokenOwner, msg.sender, tokenId);
        uint256 price = tokenPrices[tokenId];
        tokenPrices[tokenId] = 0;
        payable(tokenOwner).transfer(price);
        emit TokenSold(tokenId, msg.sender, price);
    }

    // 撤销NFT的销售
    function cancelSale(uint256 tokenId) public {
        require(_exists(tokenId), "Token does not exist");
        require(ownerOf(tokenId) == msg.sender, "Only token owner can cancel sale");
        tokenPrices[tokenId] = 0;
        tokenApprovals[tokenId] = address(0);
        emit TokenListed(tokenId, 0);
    }

    // 提取合约余额，只有合约所有者可调用
    function withdrawContractBalance() public onlyOwner {
        uint256 contractBalance = address(this).balance;
        require(contractBalance > 0, "Contract has no balance to withdraw");
        payable(owner()).transfer(contractBalance);
    }
}
```

在这个合约中，创作者可以上传各种形式的创作内容，并将其转化为NFT，这样他们就能够保护自己的作品并在市场上销售。同时，通过设置NFT的销售价格和撤销销售等功能，创作者能够灵活地管理自己的作品。