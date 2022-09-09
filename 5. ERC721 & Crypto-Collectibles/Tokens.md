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
