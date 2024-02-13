### [M=#]  An unbounded array in the `PuppyRaffle::enterRaffle` function can lead to so many entrants that the gas fees becoming exorbitant.

**Description:** When the array is looped through to check for duplicate entries, the gas fees are higher for each new player that enters the raffle. If an attacker notices this vulnerability, they could carry out a denial of service attack by populating the array with too many elements that take the gas fees past the gas block limit.

<details>
<summary>Function in question</summary>

```javascript
function enterRaffle(address[] memory newPlayers) public payable {
        require(msg.value == entranceFee * newPlayers.length, "PuppyRaffle: Must send enough to enter raffle");
        for (uint256 i = 0; i < newPlayers.length; i++) {
            players.push(newPlayers[i]);
        }

        // Check for duplicates
        // @audit - this nested loop is reading from storage n^2 times, can be optimised by assigning the
        // list of players to a local variable in function scope, or use a mapping, Do it in the for loop above before .push

        // If a bad actor continuously joins the raffle with different addresses it will cause a DoS attack and  killer GAS fees
        for (uint256 i = 0; i < players.length - 1; i++) {
            for (uint256 j = i + 1; j < players.length; j++) {
                require(players[i] != players[j], "PuppyRaffle: Duplicate player");
            }
        }
        emit RaffleEnter(newPlayers);
    }
```

</details>


**Impact** The entire functionality of the contract could be rendered uselses during a DoS attack. This could also be used to manipulate the winner of the raffle

**Proof of Concept:** A test `PuppyRaffle::testDenialOfServiceAttack` was written in the `PuppyRaffle::PuppyRaffleTest.t.sol` which calls the array with a smaller and then larger number of players, showing the disproportionale growth of gas fees to new entrants.

```bash
    Gas used for 100 players:  6252039
    Gas used for 1000 players:  500248579
```

Place the following test into `PuppyRaffleTest.t.sol`
<details>
<summary> The test to produce the output similar to above from console.log statements </summary>

```javascript
function testDenialOfServiceAttack() public {
        // set gas price
        vm.txGasPrice(1);

        // enter raffle with 1000 players
        address[] memory players = new address[](100);
        for (uint256 i = 0; i < players.length; i++) {
            players[i] = address(i);
        }

        // get the gas value before entering the raffle
        uint256 gasStart = gasleft();
        // vm.expectRevert();
        puppyRaffle.enterRaffle{value: entranceFee * 100}(players);

        // ge the gas value after entering the raffle
        uint256 gasEnd = gasleft();

        uint256 gasUsedFirst = gasStart - gasEnd;

        console.log("Gas used for 100 players: ", gasUsedFirst);

                // enter raffle with 1000 players
        address[] memory players1000 = new address[](1000);
        for (uint256 i = 0; i < players1000.length; i++) {
            players1000[i] = address(i+players.length);
        }

        // get the gas value before entering the raffle
        uint256 gasStart1000 = gasleft();
        // vm.expectRevert();
        puppyRaffle.enterRaffle{value: entranceFee * 1000}(players1000);

        // ge the gas value after entering the raffle
        uint256 gasEnd1000 = gasleft();

        uint256 gasUsedFirst1000 = gasStart1000 - gasEnd1000;

        console.log("Gas used for 1000 players: ", gasUsedFirst1000);
    }
```

</details>

The gas fees are over 100x higher with 1000 players instead of 100, which is 10x more entrants. 

**Recommended Mitigation:** Use a mapping of addresses to entrants in the raffle instead of looping through the array. Create a maximum length for the array of entrants.

