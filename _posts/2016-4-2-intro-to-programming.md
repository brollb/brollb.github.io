---
layout: post
title:  "Intro to Programming with JS"
date:   2016-4-2 15:06:00
categories: javascript
---

I really like [learn x in y minutes](https://learnxinyminutes.com/). I think it is very effective at creating a very condensed, thorough example demonstrating many features of programming languages.

Recently, I was helping a friend get introduced to programming and I took an approach similar to explain javascript for someone with no programming background. This file is far from complete but I thought I would share it in the hopes that others could find it useful to introduce people to computer programming using javascript:

```javascript
////// Variables ////// 
// variables are created using "var" keyword
var word = 'hello';

// variables behave like they do in math; they are a placeholder for their value
//
// In our case, the variable "word" is gonna be replaced with "hello"
word;  // same as "hello"

// JS also assumes that any word that you write that is not a keyword (like "var")
// is a variable
//
//asdf;  // fails
'asdf';  // now js knows it is text

////// Functions ////// 
// Functions are instructions/procedure. Essentially, they do something
// functions are created using "function"
//
// Here is a simple function (the simplest)
//
//function () {
//}

// JS supports "functions as data". So we can store a function in/as variables
//
// For example, if we want a function called "hello":
var hello = function() {
};

// Functions, unlike other data types (words, numbers, etc), can be "called" or
// "invoked" using ()
hello();  // runs the function stored as hello


// console.log is a function that exists by default. It prints to the console (terminal)
console.log('hello world');

// in the above example, 'console.log' is invoked with the "argument" "hello world"
//
// an argument is something a function needs to be able to do it's job (or run)
//
// we can define a function that needs arguments by add them between the () when
// we define/create the function
hello = function(cat) {
    console.log(cat);
};

// now "hello" is a function that, given an argument "cat", it will print this
// value using "console.log"
hello('world');

// when "hello" is called with "world", it sets it's "cat" argument to "world"
//
//hello = function(cat) {
    //cat is now "world"; ie, cat = 'world'
    //console.log(cat);
//};

// like, in math, when you define a function
//
//   f(x) = 2 * x
//
// when we call f(3), we go back to the definition and replace all x's with 3's
//
//   f(3) = 2 * 3
//
// now we know that f(3) = 6
//
//
// Programming works similarly. In the hello example, we called it with "world"
// so we basically are replacing all "cat" variables with "world"

// now let's make another function (multiple variables!)
var add = function(x, y) {
    console.log(x + y);
};


// now, add is a function that prints the sum of two numbers!
add(3, 5);

// Another functions can do is "return values". The problem with our add function
// is that it prints the value - what if we didn't want to print it out.
//
// the keyword "return" returns a value

// let's update this add function
add = function(x, y) {
    return x + y;
};

// now add won't print to the console
add(5, 6);

// we can now say after the fact that we want to print the answer
console.log(add(5, 6));

// but we can now store the result in a new variable
var sum = add(10, 40);

console.log(sum);

sum = add(sum, 20);

console.log(sum);

///// Lists /////

// in JS, sometimes you want to store a list of information. This is done using []
// and putting the values inside separated with commas
var list = [1, 2, 3];

console.log(list);

// lists have a value called length. This can be retrieved by typing '.length'
// after the list variable
console.log(list.length);

// Retrieving things from lists
// This is done using [] right after the list variable with the number/index of
// the thing out of it

console.log(list[2]);

// One gotcha is that the elements are zero indexed - counting is 0, 1, 2, ... 
// NOT 1, 2, 3...

// Replacing values in lists
//
// As you might expect, you just use the same syntax as gettting values but use
// '=' after

list[2] = 100;

console.log(list);

// adding and removing from lists
//
// add elements to the back of the list with '.push()'

list.push(17);

console.log(list);
console.log(list.length);

// remove elements from the back with '.pop()'

var last = list.pop();

console.log(list);

console.log(last);

// on an aside, you can combine text using '+' and everything can be turned to
// text (though it doesn't always look pretty)
console.log('my final list is ' + list);

// with console.log, you can also pass multiple arguments (with commas between)
// to print more intelligently

console.log('my final list is', list);

///// Boolean /////

// boolean values are true or false
var bool = true;

console.log('bool is', bool);

// we can also compare things and check stuff with >, <, ==, !=, ===, !==
console.log('2 === 3?', 2 === 3);

// ! inverts it. !=== means not equal
console.log('2 !== 3?', 2 !== 3);

console.log('!bool', !bool);
console.log('!!bool', !!bool);

///// Objects /////

// objects are things that have fields that contain stuff
var bob = {};  // bob is now an object
console.log('bob is', bob);

// I can add fields to an object using '.field_name' (like '.length' w/ lists)
bob.name = 'bob';

console.log('bob now has a name', bob);


// we can also add lists, functions, etc
bob.age = 40;

bob.sayHi = function() {
    console.log('hi!');
};

console.log('bob should say hi');
bob.sayHi();

// when an object calls a function (like bob.sayHi), the function has a "this"
// value which refers to the caller (in this case, bob);

bob.introduce = function() {
    // since bob is gonna be the caller of this function,
    // 'this' is bob himself. this.name is like 'bob.name' above
    console.log('hi, my name is', this.name);
};

bob.introduce();

// Objects can also be created using a 'literal notation'. Bob, in literal notation
// would look like:
bob = {
    name: 'bob',
    age: 40,
    introduce: function() {
        console.log('hi, my name is', this.name);
    }
};

// just to prove it:
console.log('literal bob');
console.log(bob);
bob.introduce();

///// using 'require' /////
// You can use other libraries/utilities using the 'require' keyword

var path = require('path');

// path is a helper for creating file system paths like /home/irishninja/...'
// it has a function called 'join' that you can use to create these paths
//
// using path.join creates the path url across platforms. In other words foo/bar
// on linux and mac but foo\bar on windows
var myPath = path.join('this', 'is', 'my', 'path');

console.log(myPath);

// most importantly this is just demonstrating how to use other libraries, etc,
// using nodejs. This is different in the browser
//

///// loops /////
// to iterate over lists, etc. basically to do things over and over for a while,
// js has two main types of loops: 'while' an 'for' loops
//
// 'while' loops are probably simpler:
//
// let's add all numbers in the following list
console.log('/////////////////////// loops ///////////////////////');
var list = [1, 2, 3, 4];
var sum = 0;

// basically, we want
sum = list[0] + list[1] + list[2] + list[3];

console.log('sum is', sum);
sum = 0;

// but this sucks because it only works for one size of list. What if we don't
// know the length. This is where we use a loop
var index = 0;

while (index < list.length) {
    console.log('adding', list[index], 'to', sum);
    sum = sum + list[index];
    index = index + 1;
    // same as:
    //   index++;
    //   index += 1;
}

// this means that while the index is less than the length of the list, it will
// get the number at that spot and add it to the sum. Then it increases the index
console.log('sum is', sum);

// To convert a text number to an actual number, use +
console.log('2' + '3');

console.log(Number('2') + Number('3'));

// or
console.log((+'2') + (+'3'));
console.log(parseInt('2') + parseInt(+'3'));


// converting a while loop to a for loop
var index = 0;  // this is first
while (index < list.length /* this is second */) { 
    sum = sum + list[index];
    index = index + 1;  // this is last
}

// so...
for (var index = 0; index < list.length; index = index + 1) {
    sum = sum + list[index];
}

// same as the while loop as far as behavior - just a different way to write it

///// if statements /////
// if statements are like a while loop but replace 'while' with 'if' and it runs
// only once

if (index < list.length) {
    sum = sum + list[index];
} else if (index === list.length) {
    // index is the same as the length
} else {
    // execute this stuff if index > list.length
}
```
