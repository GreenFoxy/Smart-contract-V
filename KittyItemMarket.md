
https://etherscan.io/address/0xfc9ec868f4c8c586d1bb7586870908cca53d5f38#code

Integer Overflow

      function buyItem(string _itemName, uint256 _amount) public payable {
        require(paused == false);
        require(items[_itemName].itemContract != 0x0);  // item should already exist
        Item storage item = items[_itemName];  // we're going to modify the item in storage
        require(msg.value >= item.cost * _amount);  // make sure user sent enough eth for the number of items they want
        item.totalFunds += msg.value;
        KittyItemToken kit = KittyItemToken(item.itemContract);
        kit.transfer(msg.sender, _amount);
        // emit events
        Buy(_itemName);
      }
      
There exists a integer overflow at line 5 `require(msg.value >= item.cost * _amount);`.

For example, if one of the `item`'s `item.cost` is set to 100, and someone call this function with `_amount` = 2^256/100, then the result of mul operation at line 5 will be 0, and the assertion of `(msg.value >= item.cost * _amount)` will always stand.
 
That means, a with carefully constructed `_amount`, you can bypass the check at line 5 with only few or even 0 ether! And as a result, it will transfer the tokens in the contract at address `item.itemContract` hold by itself to you with the amount of `_amount`(line 7,8).
