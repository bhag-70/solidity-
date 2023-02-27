// SPDX-License-Identifier: GPL-3.0

pragma solidity ^0.8.0;

contract PaintingMarketplace {
    struct Painting {
        uint256 id;
        address owner;
        string name;
        string artist;
        uint256 price;
        bool isSold;
        address buyer;
    }

    Painting[] public paintings;

    mapping(uint256 => address) public paintingToOwner;
    mapping(address => uint256) public ownerPaintingCount;

    event PaintingCreated(uint256 id, address owner, string name, string artist, uint256 price);
    event PaintingSold(uint256 id, address buyer);

    function createPainting(string memory _name, string memory _artist, uint256 _price) public {
        require(_price > 0, "Price should be greater than 0");

        uint256 id = paintings.length;
        paintings.push(Painting(id, msg.sender, _name, _artist, _price, false, address(0)));
        paintingToOwner[id] = msg.sender;
        ownerPaintingCount[msg.sender]++;

        emit PaintingCreated(id, msg.sender, _name, _artist, _price);
    }

    function buyPainting(uint256 _id) public payable {
        Painting storage painting = paintings[_id];
        require(!painting.isSold, "Painting has already been sold");
        require(msg.value == painting.price, "Insufficient funds");

        painting.isSold = true;
        painting.buyer = msg.sender;
        ownerPaintingCount[painting.owner]--;
        ownerPaintingCount[msg.sender]++;
        paintingToOwner[_id] = msg.sender;

        emit PaintingSold(_id, msg.sender);
    }

    function getPainting(uint256 _id) public view returns (
        uint256 id,
        address owner,
        string memory name,
        string memory artist,
        uint256 price,
        bool isSold,
        address buyer
    ) {
        Painting memory painting = paintings[_id];
        return (
            painting.id,
            painting.owner,
            painting.name,
            painting.artist,
            painting.price,
            painting.isSold,
            painting.buyer
        );
    }
}
