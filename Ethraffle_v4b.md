Ethraffle_v4b
---------
https://etherscan.io/address/0xcC88937F325d1C6B97da0AFDbb4cA542EFA70870#code

Bad randomness

    function chooseWinner() private {
        address seed1 = contestants[uint(block.coinbase) % totalTickets].addr;
        address seed2 = contestants[uint(msg.sender) % totalTickets].addr;
        uint seed3 = block.difficulty;
        bytes32 randHash = keccak256(seed1, seed2, seed3);

        uint winningNumber = uint(randHash) % totalTickets;
        address winningAddress = contestants[winningNumber].addr;
        
        ......
        
        winningAddress.transfer(prize);
    }

