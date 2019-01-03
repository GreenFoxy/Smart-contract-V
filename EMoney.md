
https://etherscan.io/address/0x01fa9880b33e6c00eafc866a48d292dbe07b7578#code

Integer Overflow

    function setPrices(uint256 newSellPrice, uint256 newBuyPrice) onlyOwner public {
        sellPrice = newSellPrice;
        buyPrice = newBuyPrice;
    }
    function sell(uint256 amount) public {
        require(this.balance &gt;= amount * sellPrice);     
        _transfer(msg.sender, this, amount);              
        msg.sender.transfer(amount * sellPrice);          
    }
      

There exists a integer overflow in function sell().

If the owner set the value of `sellPrice` to a large number like 2^255 in setPrices(), then the `(amount * sellPrice)` in `sell()` will cause an integer overflow and finally the user calling `sell()` to sell tokens will get ethers much less than expeted if `amount > 1`.
