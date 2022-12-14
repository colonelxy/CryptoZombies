## Chapter 1: Tokens on Ethereum

Let's talk about tokens.

If you've been in the Ethereum space for any amount of time, you've probably heard people talking about tokens — specifically ERC20 tokens.

A token on Ethereum is basically just a smart contract that follows some common rules — namely it implements a standard set of functions that all other token contracts share, such as transferFrom(address _from, address _to, uint256 _tokenId) and balanceOf(address _owner).

Internally the smart contract usually has a mapping, mapping(address => uint256) balances, that keeps track of how much balance each address has.

So basically a token is just a contract that keeps track of who owns how much of that token, and some functions so those users can transfer their tokens to other addresses.
Why does it matter?

Since all ERC20 tokens share the same set of functions with the same names, they can all be interacted with in the same ways.

This means if you build an application that is capable of interacting with one ERC20 token, it's also capable of interacting with any ERC20 token. That way more tokens can easily be added to your app in the future without needing to be custom coded. You could simply plug in the new token contract address, and boom, your app has another token it can use.

One example of this would be an exchange. When an exchange adds a new ERC20 token, really it just needs to add another smart contract it talks to. Users can tell that contract to send tokens to the exchange's wallet address, and the exchange can tell the contract to send the tokens back out to users when they request a withdraw.

The exchange only needs to implement this transfer logic once, then when it wants to add a new ERC20 token, it's simply a matter of adding the new contract address to its database.
Other token standards

ERC20 tokens are really cool for tokens that act like currencies. But they're not particularly useful for representing zombies in our zombie game.

For one, zombies aren't divisible like currencies — I can send you 0.237 ETH, but transfering you 0.237 of a zombie doesn't really make sense.

Secondly, all zombies are not created equal. Your Level 2 zombie "Steve" is totally not equal to my Level 732 zombie "H4XF13LD MORRIS 💯💯😎💯💯". (Not even close, Steve).

There's another token standard that's a much better fit for crypto-collectibles like CryptoZombies — and they're called `ERC721` tokens.

ERC721 tokens are not interchangeable since each one is assumed to be unique, and are not divisible. You can only trade them in whole units, and each one has a unique ID. So these are a perfect fit for making our zombies tradeable.

    Note that using a standard like ERC721 has the benefit that we don't have to implement the auction or escrow logic within our contract that determines how players can trade / sell our zombies. If we conform to the spec, someone else could build an exchange platform for crypto-tradable ERC721 assets, and our ERC721 zombies would be usable on that platform. So there are clear benefits to using a token standard instead of rolling your own trading logic.



We're going to dive into the ERC721 implementation in the next chapter. But first, let's set up our file structure for this lesson.

We're going to store all the ERC721 logic in a contract called ZombieOwnership.

    Declare our `pragma version` at the top of the file (check previous lessons' files for the syntax).

    This file should import from `zombieattack.sol`.

    Declare a new contract, `ZombieOwnership`, that inherits from ZombieAttack. Leave the body of the contract empty for now.





    pragma solidity >=0.5.0 <0.6.0;
    import "./zombieattack.sol";

    contract ZombieOwnership is ZombieAttack {
        
    }

## Chapter 2: ERC721 Standard, Multiple Inheritance

Let's take a look at the ERC721 standard:

    contract ERC721 {
    event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
    event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);

    function balanceOf(address _owner) external view returns (uint256);
    function ownerOf(uint256 _tokenId) external view returns (address);
    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
    function approve(address _approved, uint256 _tokenId) external payable;
    }

This is the list of methods we'll need to implement, which we'll be doing over the coming chapters in pieces.

It looks like a lot, but don't get overwhelmed! We're here to walk you through it.
Implementing a token contract

When implementing a token contract, the first thing we do is copy the interface to its own Solidity file and import it, import "./erc721.sol";. Then we have our contract inherit from it, and we override each method with a function definition.

But wait — ZombieOwnership is already inheriting from ZombieAttack — how can it also inherit from ERC721?

Luckily in Solidity, your contract can inherit from multiple contracts as follows:

    contract SatoshiNakamoto is NickSzabo, HalFinney {
    // Omg, the secrets of the universe revealed!
    }

As you can see, when using multiple inheritance, you just separate the multiple contracts you're inheriting from with a comma, ,. In this case, our contract is inheriting from NickSzabo and HalFinney.

Let's give it a try.

We've already created erc721.sol with the interface above for you.

    Import erc721.sol into zombieownership.sol

    Declare that ZombieOwnership inherits from ZombieAttack AND ERC721





    import "./erc721.sol";

    contract ZombieOwnership is ZombieAttack, ERC721 {

    }


## Chapter 3: balanceOf & ownerOf

Great, let's dive into the ERC721 implementation!

We've gone ahead and copied the empty shell of all the functions you'll be implementing in this lesson.

In this chapter, we're going to implement the first two methods: balanceOf and ownerOf.

balanceOf

    function balanceOf(address _owner) external view returns (uint256 _balance);

This function simply takes an address, and returns how many tokens that address owns.

In our case, our "tokens" are Zombies. Do you remember where in our DApp we stored how many zombies an owner has?
ownerOf

    function ownerOf(uint256 _tokenId) external view returns (address _owner);

This function takes a token ID (in our case, a Zombie ID), and returns the address of the person who owns it.

Again, this is very straightforward for us to implement, since we already have a mapping in our DApp that stores this information. We can implement this function in one line, just a return statement.

    Note: Remember, uint256 is equivalent to uint. We've been using uint in our code up until now, but we're using uint256 here because we copy/pasted from the spec.



I'll leave it to you to figure out how to implement these 2 functions.

Each function should simply be 1 line of code, a return statement. Take a look at our code from previous lessons to see where we're storing this data.

- Implement balanceOf to return the number of zombies _owner has.

- Implement ownerOf to return the address of whoever owns the zombie with ID _tokenId.

    return ownerZombieCount[_owner];

    return zombieToOwner[_tokenId];

## Chapter 4: Refactoring

Uh oh! We've just introduced an error in our code that will make it not compile. Did you notice it?

In the previous chapter we defined a function called ownerOf. But if you recall from Lesson 4, we also created a modifier with the same name, ownerOf, in zombiefeeding.sol.

If you tried compiling this code, the compiler would give you an error saying you can't have a modifier and a function with the same name.

So should we just change the function name in ZombieOwnership to something else?

No, we can't do that!!! Remember, we're using the ERC721 token standard, which means other contracts will expect our contract to have functions with these exact names defined. That's what makes these standards useful — if another contract knows our contract is ERC721-compliant, it can simply talk to us without needing to know anything about our internal implementation decisions.

So that means we'll have to refactor our code from Lesson 4 to change the name of the modifier to something else.

We're back in zombiefeeding.sol. We're going to change the name of our modifier from ownerOf to onlyOwnerOf.

- Change the name of the modifier definition to onlyOwnerOf

- Scroll down to the function feedAndMultiply, which uses this modifier. We'll need to change the name here as well.

- We also use this modifier in zombiehelper.sol and zombieattack.sol.

    modifier onlyOwnerOf(uint _zombieId) {
        require(msg.sender == zombieToOwner[_zombieId]);
        _;
    }

    function feedAndMultiply(uint _zombieId, uint _targetDna, string memory _species) internal onlyOwnerOf(_zombieId) { 
        }

## Chapter 5: ERC721: Transfer Logic

Great, we've fixed the conflict!

Now we're going to continue our ERC721 implementation by looking at transfering ownership from one person to another.

Note that the ERC721 spec has 2 different ways to transfer tokens:

    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;

and

    function approve(address _approved, uint256 _tokenId) external payable;

    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;

- The first way is the token's owner calls transferFrom with his address as the _from parameter, the address he wants to transfer to as the _to parameter, and the _tokenId of the token he wants to transfer.

- The second way is the token's owner first calls approve with the address he wants to transfer to, and the _tokenID . The contract then stores who is approved to take a token, usually in a mapping (uint256 => address). Then, when the owner or the approved address calls transferFrom, the contract checks if that msg.sender is the owner or is approved by the owner to take the token, and if so it transfers the token to him.

Notice that both methods contain the same transfer logic. In one case the sender of the token calls the transferFrom function; in the other the owner or the approved receiver of the token calls it.

So it makes sense for us to abstract this logic into its own private function, _transfer, which is then called by transferFrom.

Let's define the logic for _transfer.

- Define a function named _transfer. It will take 3 arguments, address _from, address _to, and uint256 _tokenId. It should be a private function.

- We have 2 mappings that will change when ownership changes: ownerZombieCount (which keeps track of how many zombies an owner has) and zombieToOwner (which keeps track of who owns what).

- The first thing our function should do is increment ownerZombieCount for the person receiving the zombie (address _to). Use ++ to increment.

- Next, we'll need to decrease the ownerZombieCount for the person sending the zombie (address _from). Use -- to decrement.

- We'll want to change zombieToOwner mapping for this _tokenId so it now points to _to.

- The ERC721 spec includes a Transfer event. The last line of this function should fire Transfer with the correct information — check erc721.sol to see what arguments it's expecting to be called with and implement it here.

    function _transfer(address _from, address _to, uint256 _tokenId) private {
        ownerZombieCount[_to]++;
        ownerZombieCount[_from]--;
        zombieToOwner[_tokenId] = _to;
        emit Transfer(_from, _to, _tokenId);
    }

## Chapter 6: ERC721: Transfer Cont'd

Great! That was the difficult part — now implementing the external transferFrom function will be easy, since our _transfer function already does almost all the heavy lifting.


- First, we want to make sure only the owner or the approved address of a token/zombie can transfer it. Let's define a mapping called zombieApprovals. It should map a uint to an address. This way, when someone that is not the owner calls transferFrom with a _tokenId, we can use this mapping to quickly look up if he is approved to take that token.

- Next, let's add a require statement to transferFrom. It should make sure that only the owner or the approved address of a token/zombie can transfer it.

- Lastly, don't forget to call _transfer.

    Note: Checking that only the owner or the approved address of a token/zombie can transfer it means that at least one of these conditions must be true:

    zombieToOwner for _tokenId is equal to msg.sender

    or

    zombieApprovals for _tokenId is equal to msg.sender

Don't worry about filling in data in the zombieApprovals mapping, we'll do it in the next chapter.



    mapping (uint => address) zombieApprovals;

    function transferFrom(address _from, address _to, uint256 _tokenId) external payable {
        require (zombieToOwner[_tokenId] == msg.sender || zombieApprovals[_tokenId] == msg.sender);
        _transfer(_from, _to, _tokenId);
    }

## Chapter 7: ERC721: Approve

Now, let's implement approve.

Remember, with approve the transfer happens in 2 steps:

- You, the owner, call approve and give it the _approved address of the new owner, and the _tokenId you want them to take.

- The new owner calls transferFrom with the _tokenId. Next, the contract checks to make sure the new owner has been already approved, and then transfers them the token.

Because this happens in 2 function calls, we need to use the zombieApprovals data structure to store who's been approved for what in between function calls.
Putting it to the Test

- In the `approve` function, we want to make sure only the owner of the token can give someone approval to take it. So we need to add the `onlyOwnerOf` modifier to approve

- For the body of the function, set `zombieApprovals` for `_tokenId` equal to the `_approved` address.



function approve(address _approved, uint256 _tokenId) external payable onlyOwnerOf(_tokenId) {
    zombieApprovals[_tokenId] = _approved;
  }

## Chapter 8: ERC721: Approve

Great, we are almost done!

There is one more thing to do- there's an Approval event in the ERC721 spec. So we should fire this event at the end of the approve function.
Putting it to the Test

- Let's fire the Approval event. Take a look at the erc721.sol file for the arguments, and be sure to use msg.sender as _owner.

Great, we have finished our ERC721 implementation!

    emit Approval(msg.sender, _approved, _tokenId);

## Chapter 9: Preventing Overflows

Congratulations, that completes our ERC721 and ERC721x implementation!

That wasn't so tough, was it? A lot of this Ethereum stuff sounds really complicated when you hear people talking about it, so the best way to understand it is to actually go through an implementation of it yourself.

Keep in mind that this is only a minimal implementation. There are extra features we may want to add to our implementation, such as some extra checks to make sure users don't accidentally transfer their zombies to address 0 (which is called "burning" a token — basically it's sent to an address that no one has the private key of, essentially making it unrecoverable). Or to put some basic auction logic in the DApp itself. (Can you think of some ways we could implement that?)

But we wanted to keep this lesson manageable, so we went with the most basic implementation. If you want to see an example of a more in-depth implementation, you can take a look at the OpenZeppelin ERC721 contract after this tutorial.
Contract security enhancements: Overflows and Underflows

We're going to look at one major security feature you should be aware of when writing smart contracts: Preventing overflows and underflows.

What's an overflow?

Let's say we have a uint8, which can only have 8 bits. That means the largest number we can store is binary 11111111 (or in decimal, 2^8 - 1 = 255).

Take a look at the following code. What is number equal to at the end?

uint8 number = 255;
number++;

In this case, we've caused it to overflow — so number is counterintuitively now equal to 0 even though we increased it. (If you add 1 to binary 11111111, it resets back to 00000000, like a clock going from 23:59 to 00:00).

An underflow is similar, where if you subtract 1 from a uint8 that equals 0, it will now equal 255 (because uints are unsigned, and cannot be negative).

While we're not using uint8 here, and it seems unlikely that a uint256 will overflow when incrementing by 1 each time (2^256 is a really big number), it's still good to put protections in our contract so that our DApp never has unexpected behavior in the future.
Using SafeMath

To prevent this, OpenZeppelin has created a library called SafeMath that prevents these issues by default.

But before we get into that... What's a library?

A library is a special type of contract in Solidity. One of the things it is useful for is to attach functions to native data types.

For example, with the SafeMath library, we'll use the syntax using SafeMath for uint256. The SafeMath library has 4 functions — add, sub, mul, and div. And now we can access these functions from uint256 as follows:

using SafeMath for uint256;

uint256 a = 5;
uint256 b = a.add(3); // 5 + 3 = 8
uint256 c = a.mul(2); // 5 * 2 = 10

We'll look at what these functions do in the next chapter, but for now let's add the SafeMath library to our contract.
Putting it to the Test

We've already included OpenZeppelin's SafeMath library for you in safemath.sol. You can take a quick peek at the code now if you want to, but we'll be looking at it in depth in the next chapter.

First let's tell our contract to use SafeMath. We'll do this in ZombieFactory, our very base contract — that way we can use it in any of the sub-contracts that inherit from this one.

    Import safemath.sol into zombiefactory.sol.

    Add the declaration using SafeMath for uint256;.



    import "./safemath.sol";

contract ZombieFactory is Ownable {

  using SafeMath for uint256;
  }

## Chapter 10: SafeMath Part 2

Let's take a look at the code behind SafeMath:

library SafeMath {

  function mul(uint256 a, uint256 b) internal pure returns (uint256) {
    if (a == 0) {
      return 0;
    }
    uint256 c = a * b;
    assert(c / a == b);
    return c;
  }

  function div(uint256 a, uint256 b) internal pure returns (uint256) {
    // assert(b > 0); // Solidity automatically throws when dividing by 0
    uint256 c = a / b;
    // assert(a == b * c + a % b); // There is no case in which this doesn't hold
    return c;
  }

  function sub(uint256 a, uint256 b) internal pure returns (uint256) {
    assert(b <= a);
    return a - b;
  }

  function add(uint256 a, uint256 b) internal pure returns (uint256) {
    uint256 c = a + b;
    assert(c >= a);
    return c;
  }
}

First we have the library keyword — libraries are similar to contracts but with a few differences. For our purposes, libraries allow us to use the using keyword, which automatically tacks on all of the library's methods to another data type:

using SafeMath for uint;
// now we can use these methods on any uint
uint test = 2;
test = test.mul(3); // test now equals 6
test = test.add(5); // test now equals 11

Note that the mul and add functions each require 2 arguments, but when we declare using SafeMath for uint, the uint we call the function on (test) is automatically passed in as the first argument.

Let's look at the code behind add to see what SafeMath does:

function add(uint256 a, uint256 b) internal pure returns (uint256) {
  uint256 c = a + b;
  assert(c >= a);
  return c;
}

Basically add just adds 2 uints like +, but it also contains an assert statement to make sure the sum is greater than a. This protects us from overflows.

assert is similar to require, where it will throw an error if false. The difference between assert and require is that require will refund the user the rest of their gas when a function fails, whereas assert will not. So most of the time you want to use require in your code; assert is typically used when something has gone horribly wrong with the code (like a uint overflow).

So, simply put, SafeMath's add, sub, mul, and div are functions that do the basic 4 math operations, but throw an error if an overflow or underflow occurs.
Using SafeMath in our code.

To prevent overflows and underflows, we can look for places in our code where we use +, -, *, or /, and replace them with add, sub, mul, div.

Ex. Instead of doing:

myUint++;

We would do:

myUint = myUint.add(1);

Putting it to the Test

We have 2 places in ZombieOwnership where we used math operations. Let's swap them out with SafeMath methods.

    Replace ++ with a SafeMath method.

    Replace -- with a SafeMath method.

    ownerZombieCount[_to] = ownerZombieCount[_to].add(1);
    ownerZombieCount[_from] = ownerZombieCount[_from].sub(1);

## Chapter 11: SafeMath Part 3

Great, now our ERC721 implementation is safe from overflows & underflows!

Going back through the code we wrote in previous lessons, there's a few other places in our code that could be vulnerable to overflows or underflows.

For example, in ZombieAttack we have:

myZombie.winCount++;
myZombie.level++;
enemyZombie.lossCount++;

We should prevent overflows here as well just to be safe. (It's a good idea in general to just use SafeMath instead of the basic math operations. Maybe in a future version of Solidity these will be implemented by default, but for now we have to take extra security precautions in our code).

However we have a slight problem — winCount and lossCount are uint16s, and level is a uint32. So if we use SafeMath's add method with these as arguments, it won't actually protect us from overflow since it will convert these types to uint256:

function add(uint256 a, uint256 b) internal pure returns (uint256) {
  uint256 c = a + b;
  assert(c >= a);
  return c;
}

// If we call `.add` on a `uint8`, it gets converted to a `uint256`.
// So then it won't overflow at 2^8, since 256 is a valid `uint256`.

This means we're going to need to implement 2 more libraries to prevent overflow/underflows with our uint16s and uint32s. We can call them SafeMath16 and SafeMath32.

The code will be exactly the same as SafeMath, except all instances of uint256 will be replaced with uint32 or uint16.

We've gone ahead and implemented that code for you — go ahead and look at safemath.sol to see the code.

Now we need to implement it in ZombieFactory.
Putting it to the Test

Assignment:

    Declare that we're using SafeMath32 for uint32.

    Declare that we're using SafeMath16 for uint16.

    There's one more line of code in ZombieFactory where we should use a SafeMath method. In the function _createZombie under zombie,push().

using SafeMath for uint256;
using SafeMath32 for uint32;
using SafeMath16 for uint16;

ownerZombieCount[msg.sender] = ownerZombieCount[msg.sender].add(1);

## Chapter 12: SafeMath Part 4

Great, now we can implement SafeMath on all the types of uints we used in our DApp!

Let's fix all those potential issues in ZombieAttack. (There was also one zombies[_zombieId].level++; that needed to be fixed in ZombieHelper, but we've taken care of that one for you so we don't take an extra chapter to do so 😉).
Putting it to the Test

Go ahead and implement SafeMath methods on all the ++ increments in ZombieAttack. 

 randNonce = randNonce.add(1);

 if (rand <= attackVictoryProbability) {
      myZombie.winCount = myZombie.winCount.add(1);
      myZombie.level = myZombie.level.add(1);
      enemyZombie.lossCount = enemyZombie.lossCount.add(1);
      feedAndMultiply(_zombieId, enemyZombie.dna, "zombie");
    } else {
      myZombie.lossCount = myZombie.lossCount.add(1);
      enemyZombie.winCount = enemyZombie.winCount.add(1);
      _triggerCooldown(myZombie);
    }

## Chapter 13: Comments

The Solidity code for our zombie game is finally finished!

In the next lessons, we'll look at how to deploy the code to Ethereum, and how to interact with it with Web3.js.

But one final thing before we let you go in Lesson 5: Let's talk about commenting your code.
Syntax for comments

Commenting in Solidity is just like JavaScript. You've already seen some examples of single line comments throughout the CryptoZombies lessons:

// This is a single-line comment. It's kind of like a note to self (or to others)

Just add double // anywhere and you're commenting. It's so easy that you should do it all the time.

But I hear you — sometimes a single line is not enough. You are born a writer, after all!

Thus we also have multi-line comments:

contract CryptoZombies {
  /* This is a multi-lined comment. I'd like to thank all of you
    who have taken your time to try this programming course.
    I know it's free to all of you, and it will stay free
    forever, but we still put our heart and soul into making
    this as good as it can be.

    Know that this is still the beginning of Blockchain development.
    We've come very far but there are so many ways to make this
    community better. If we made a mistake somewhere, you can
    help us out and open a pull request here:
    https://github.com/loomnetwork/cryptozombie-lessons

    Or if you have some ideas, comments, or just want to say
    hi - drop by our Telegram community at https://t.me/loomnetworkdev
  */
}

In particular, it's good practice to comment your code to explain the expected behavior of every function in your contract. This way another developer (or you, after a 6 month hiatus from a project!) can quickly skim and understand at a high level what your code does without having to read the code itself.

The standard in the Solidity community is to use a format called natspec, which looks like this:

/// @title A contract for basic math operations
/// @author H4XF13LD MORRIS 💯💯😎💯💯
/// @notice For now, this contract just adds a multiply function
contract Math {
  /// @notice Multiplies 2 numbers together
  /// @param x the first uint.
  /// @param y the second uint.
  /// @return z the product of (x * y)
  /// @dev This function does not currently check for overflows
  function multiply(uint x, uint y) returns (uint z) {
    // This is just a normal comment, and won't get picked up by natspec
    z = x * y;
  }
}

@title and @author are straightforward.

@notice explains to a user what the contract / function does. @dev is for explaining extra details to developers.

@param and @return are for describing what each parameter and return value of a function are for.

Note that you don't always have to use all of these tags for every function — all tags are optional. But at the very least, leave a @dev note explaining what each function does.

Give it a try anyway, and try adding some natspec tags to ZombieOwnership:

    @title — E.g. A contract that manages transfering zombie ownership

    @author — Your name!

    @dev — E.g. Compliant with OpenZeppelin's implementation of the ERC721 spec draft



/// @title - a contract that manages zombie ownership.
/// @author - Harold
/// @dev - The code is compliant with OpenZepplin's implementation of the ERC721 spec draft.

## Chapter 14: Wrapping It Up

Congratulations! That concludes Lesson 5.

Let's recap:

In this lesson we learned about:

    Tokens, the ERC721 standard, and tradable assets/zombies
    Libraries and how to use them
    How to prevent overflows and underflows using the SafeMath library
    Commenting your code and the natspec standard

This lesson concludes our game's Solidity code!

In the next 2 lessons, we're going to look at how to deploy your contracts and interact with them using web3.js (so you can build a front-end for your DApp).


