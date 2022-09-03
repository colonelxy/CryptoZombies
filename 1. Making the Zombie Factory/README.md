# Crypto Zombie

This course will teach you how to build a game on Ethereum. It's dapted from the [CryptoZombie]('https://cryptozombies.io/en/lesson/1/') course.

It's designed for beginners to Solidity, but it assumes you have some experience programming in another language (e.g. Javascript).
It's designed for beginners to use the game.

## Chapter 1: Lesson Overview

In Lesson 1, you're going to build a "Zombie Factory" to build an army of zombies.

    Our factory will maintain a database of all zombies in our army
    Our factory will have a function for creating new zombies
    Each zombie will have a random and unique appearance

## How Zombie DNA Works

The zombie's appearance will be based on its "Zombie DNA". Zombie DNA is simple — it's a 16-digit integer, like:

8356281049284737

Just like real DNA, different parts of this number will map to different traits. The first 2 digits map to the zombie's head type, the second 2 digits to the zombie's eyes, etc.

    Note: For this tutorial, we've kept things simple, and our zombies can have only 7 different types of heads (even though 2 digits allow 100 possible options). Later on we could add more head types if we wanted to increase the number of zombie variations.

For example, the first 2 digits of our example DNA above are 83. To map that to the zombie's head type, we do 83 % 7 + 1 = 7. So this Zombie would have the 7th zombie head type.

In the panel to the right, go ahead and move the head gene slider to the 7th head (the Santa hat) to see what trait the 83 would correspond to.

## Chapter 2: Contracts

### Starting with the absolute basics:

Solidity's code is encapsulated in contracts. A contract is the fundamental building block of Ethereum applications — all variables and functions belong to a contract, and this will be the starting point of all your projects.

### Version Pragma

All solidity source code should start with a "version pragma" — a declaration of the version of the Solidity compiler this code should use. This is to prevent issues with future compiler versions potentially introducing changes that would break your code.

For the scope of this tutorial, we'll want to be able to compile our smart contracts with any compiler version in the range of `0.5.0` (inclusive) to `0.8.0` (exclusive).

```
pragma solidity >=0.5.0 <0.8.0;
```
Let's also create the first contract called ZombieFactory

```
contract ZombieFactory {

}
```
## Chapter 3: State Variables & Integers

State variables are permanently stored in contract storage. This means they're written to the Ethereum blockchain. Think of them like writing to a DB.

### Unsigned Integers: uint

The uint data type is an unsigned integer, meaning its value must be non-negative. There's also an int data type for signed integers.

    Note: In Solidity, uint is actually an alias for uint256, a 256-bit unsigned integer. You can declare uints with less bits — uint8, uint16, uint32, etc.. But in general you want to simply use uint except in specific cases, which we'll talk about in later lessons.

Our Zombie DNA is going to be determined by a 16-digit number.

Declare a uint named dnaDigits, and set it equal to 16.

contract ZombieFactory {  insert the line below in the ZombieFactory contract above.

    ```
    uint dnaDigits = 16;
    ```
}

## Chapter 4: Math Operations

Math in Solidity is pretty straightforward. The following operations are the same as in most programming languages:

    Addition: x + y
    Subtraction: x - y,
    Multiplication: x * y
    Division: x / y
    Exponential operator (i.e. "x to the power of y", x^y) or x**y
    Modulus / remainder: x % y (for example, 13 % 5 is 3, because if you divide 5 into 13, 3 is the remainder)

To make sure our Zombie's DNA is only 16 characters, let's make another uint equal to 10^16. That way we can later use the modulus operator % to shorten an integer to 16 digits.

    Create a uint named dnaModulus, right under the dnaDigits uint and set it equal to 10 to the power of dnaDigits.

    ```
    uint dnaModulus = 10**dnaDigits;
    ```

## Chapter 5: Structs

Sometimes you need a more complex data type. For this, Solidity provides structs:

Structs allow you to create more complicated data types that have multiple properties.

    Note that we just introduced a new type, string. Strings are used for arbitrary-length UTF-8 data. Ex. string greeting = "Hello world!"

In our app, we're going to want to create some zombies! And zombies will have multiple properties, so this is a perfect use case for a struct.

    Create a struct named `Zombie`.

    Our Zombie struct will have 2 properties: `name` (a string), and `dna` (a uint).

Our full code will look like this:
```
contract ZombieFactory {

    uint dnaDigits = 16;
    uint dnaModulus = 10 ** dnaDigits;
    

    struct Zombie {
        string name;
        uint dna;
    }
}
 ```
## Chapter 6: Arrays

When you want a collection of something, you can use an array. There are two types of arrays in Solidity: fixed arrays and dynamic arrays:
fixed array has a fixed length of elements (2) like this:
```
uint[2] fixedArray;
```

Dynamic array has no fixed size and can keep growing for example:
```
uint[] dynamicArray;
```

You can also create an array of structs. Using the previous chapter's Person struct:

```
Person[] people;
 ```
Ddynamic Array, we can keep adding to it

Remember that state variables are stored permanently in the blockchain? So creating a dynamic array of structs like this can be useful for storing structured data in your contract, kind of like a database.

### Public Arrays

You can declare an array as public, and Solidity will automatically create a getter method for it. The syntax looks like:

```
Person[] public people;
```

Other contracts would then be able to read from, but not write to, this array. So this is a useful pattern for storing public data in your contract.

We're going to want to store an army of zombies in our app. And we're going to want to show off all our zombies to other apps, so we'll want it to be public.

    Create a public array of Zombie structs, and name it zombies.
Under the struct block above create `zombies` like this:

```
Zombie[] public zombies;
```

## Chapter 7: Function Declarations

A function declaration in solidity looks like the following:

```
function eatHamburgers(string memory _name, uint _amount) public {

}
```
This is a function named eatHamburgers that takes `2` parameters: a `string` and a `uint`. For now the body of the function is empty. Note that we're specifying the function visibility as public. We're also providing instructions about where the `_name` variable should be stored- in `memory`. This is required for all reference types such as arrays, structs, mappings, and strings.



Note: It's convention (but not required) to start function parameter variable names with an underscore `(_)` in order to differentiate them from global variables. We'll use that convention throughout our tutorial.

You would call this function like so:
```
eatHamburgers("vitalik", 100);
```

In our app, we're going to need to be able to create some zombies. Let's create a function for that under the Zombie[] above.

    Create a public function named createZombie. It should take two parameters: _name (a string), and _dna (a uint). Don't forget to pass the first argument by value by using the memory keyword

Leave the body empty for now — we'll fill it in later.

    function createZombie (string memory _name, uint _dna) public {
            
    }

## Chapter 8: Working With Structs and Arrays
Creating New Structs

Remember our Zombie struct in the previous example?

Now we're going to learn how to create new Persons and add them to our people array.

Note that `array.push()` adds something to the end of the array, so the elements are in the order we added them. See the following example:

    uint[] numbers;
    numbers.push(5);
    numbers.push(10);
    numbers.push(15);
numbers is now equal to [5, 10, 15]

Let's make our createZombie function do something!

    Fill in the function body so it creates a new Zombie, and adds it to the zombies array. The name and dna for the new Zombie should come from the function arguments.
    Let's do it in one line of code to keep things clean.

    zombies.push(Zombie(_name, _dna));

## Chapter 9: Private / Public Functions

In Solidity, functions are public by default. This means anyone (or any other contract) can call your contract's function and execute its code.

Obviously this isn't always desirable, and can make your contract vulnerable to attacks. Thus it's good practice to mark your functions as private by default, and then only make public the functions you want to expose to the world.

This means only other functions within our contract will be able to call this function and add to the numbers array.

As you can see, we use the keyword private after the function name. And as with function parameters, it's convention to start private function names with an underscore `(_)`.


Our contract's createZombie function is currently public by default — this means anyone could call it and create a new Zombie in our contract! Let's make it private.

    Modify createZombie so it's a private function.

## Chapter 10: More on Functions

In this chapter, we're going to learn about function return values, and function modifiers.
Return Values

To return a value from a function, the declaration looks like this:

    string greeting = "What's up dog";

    function sayHello() public returns (string memory) {
    return greeting;
    }

In Solidity, the function declaration contains the type of the return value (in this case string).
Function modifiers

The above function doesn't actually change state in Solidity — e.g. it doesn't change any values or write anything.

So in this case we could declare it as a `view` function, meaning it's only viewing the data but not modifying it:

    function sayHello() public view returns (string memory) {
    }

Solidity also contains `pure` functions, which means you're not even accessing any data in the app. Consider the following:

    function _multiply(uint a, uint b) private pure returns (uint) {
    return a * b;
    }

This function doesn't even read from the state of the app — its return value depends only on its function parameters. So in this case we would declare the function as pure.

    Note: It may be hard to remember when to mark functions as pure/view. Luckily the Solidity compiler is good about issuing warnings to let you know when you should use one of these modifiers.

We're going to want a helper function that generates a random DNA number from a string.

    Create a `private` function called `_generateRandomDna` right below the current function. It will take one parameter named `_str` (a string), and return a `uint`. Don't forget to set the data location of the _str parameter to memory.

    This function will view some of our contract's variables but not modify them, so mark it as view.

    function _generateRandomDna(string memory _str) private view returns (uint) {
            
    }

## Chapter 11: Keccak256 and Typecasting

We want our `_generateRandomDna` function to return a (semi) random uint. How can we accomplish this?

Ethereum has the hash function `keccak256` built in, which is a version of SHA3. A hash function basically maps an input into a random 256-bit hexadecimal number. A slight change in the input will cause a large change in the hash.

It's useful for many purposes in Ethereum, but for right now we're just going to use it for pseudo-random number generation.

Example:

6e91ec6b618bb462a4a6ee5aa2cb0e9cf30f7a052bb467b0ba58b8748c00d2e5

    keccak256(abi.encodePacked("aaaab"));
    
b1f078126895a1424524de5321b339ab00408010b7cf0e6ed451514981e58aa9

    keccak256(abi.encodePacked("aaaac"));

As you can see, the returned values are totally different despite only a 1 character change in the input.

### Typecasting

Sometimes you need to convert between data types. Take the following example:

    uint8 a = 5;
    uint b = 6;

below throws an error because a * b returns a uint, not uint8:

    uint8 c = a * b;

// we have to typecast b as a uint8 to make it work:

    uint8 c = a * uint8(b);

In the above, a * b returns a uint, but we were trying to store it as a uint8, which could cause potential problems. By casting it as a uint8, it works and the compiler won't throw an error.


Let's fill in the body of our `_generateRandomDna` function! Here's what it should do:

The first line of code should take the keccak256 hash of abi.encodePacked(_str) to generate a pseudo-random hexadecimal, typecast it as a uint, and finally store the result in a uint called rand.

We want our DNA to only be `16 digits` long (remember our dnaModulus?). So the second line of code should return the above value modulus (%) dnaModulus.


    function _generateRandomDna(string memory _str) private view returns (uint) {
        uint rand = uint(keccak256(abi.encodePacked(_str)));
        return rand % dnaModulus;
    }

## Chapter 12: Putting It Together

We're close to being done with our random Zombie generator! Let's create a public function that ties everything together.

We're going to create a public function that takes an input, the zombie's name, and uses the name to create a zombie with random DNA.


    Create a public function named createRandomZombie. It will take one parameter named _name (a string with the data location set to memory). (Note: Declare this function public just as you declared previous functions private)

    The first line of the function should run the _generateRandomDna function on _name, and store it in a uint named randDna.

    The second line should run the _createZombie function and pass it _name and randDna.

The solution should be 4 lines of code (including the closing } of the function).


    function createRandomZombie (string memory _name) public {
        uint randDna = _generateRandomDna(_name);
        _createZombie(_name, randDna);
    }

## Chapter 13: Events

Our contract is almost finished! Now let's add an event.

Events are a way for your contract to communicate that something happened on the blockchain to your app front-end, which can be 'listening' for certain events and take action when they happen.

Your app front-end could then listen for the event. A javascript implementation would look something like:

    YourContract.IntegersAdded(function(error, result) {
    // do something with result
    })

We want an event to let our front-end know every time a new zombie was created, so the app can display it.

    Declare an event called NewZombie. It should pass zombieId (a uint), name (a string), and dna (a uint).

    Modify the _createZombie function to fire the NewZombie event after adding the new Zombie to our zombies array.

    You're going to need the zombie's id. array.push() returns a uint of the new length of the array - and since the first item in an array has index 0, array.push() - 1 will be the index of the zombie we just added. Store the result of zombies.push() - 1 in a uint called id, so you can use this in the NewZombie event in the next line.

The first line under the contract `ZombieFactory`
    event NewZombie(uint zombieId, string name, uint dna);

Modify the _createZombie function

    function _createZombie(string memory _name, uint _dna) private {
        uint id = zombies.push(Zombie(_name, _dna)) - 1;
        emit NewZombie(id, _name, _dna);

    }

## Chapter 14: Web3.js

Our Solidity contract is complete! Now we need to write a javascript frontend that interacts with the contract.

Ethereum has a Javascript library called `Web3.js`.

In a later lesson, we'll go over in depth how to deploy a contract and set up Web3.js. But for now let's just look at some sample code for how Web3.js would interact with our deployed contract.

What our javascript then does is take the values generated in zombieDetails above, and use some browser-based javascript magic (we're using Vue.js) to swap out the images and apply CSS filters. You'll get all the code for this in a later lesson.

Go ahead — type in your name to the box on the right, and see what kind of zombie you get!


