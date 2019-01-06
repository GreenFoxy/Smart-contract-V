Ethraffle_v4b
======
https://etherscan.io/address/0xcC88937F325d1C6B97da0AFDbb4cA542EFA70870#code

Vulnerability type
------
Bad randomness

Abstract
------
A gambling smart contract implementation for Ethraffle_v4b, an Ethereum gambling game, sells tickets and after the last ticket in a round was sold randomly generates a winner from the buyers who will claim the price, generates a random value that is predictable by attacker. The developer wrote a `chooseWinner()` function that uses block coinbase and difficulty as the seed. This can be predicted by writing the same random function code in an exploit contract to determine the `winningNumber` value.

Related code
------

    function buyTickets() payable public {
        ......
        // Choose winner if we sold all the tickets
        if (nextTicket == totalTickets) {
            chooseWinner();
        }
        ......
    }
    
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

Details
------
