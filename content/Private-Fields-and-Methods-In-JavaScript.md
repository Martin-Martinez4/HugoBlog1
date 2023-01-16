---
title: "Private Fields and Methods In JavaScript"
date: 2023-01-13T19:48:18-07:00
showrss: true
taglist: true
---

## JavaScript Making Data Private

ES2022 introduced a native way to create private methods and fields.  While there being a new method of declaring private methods may suggest that the older ways of extracting the functionality could be forgotten, a lot can be learned from learning these implementations.  Learning how programmers implemented private methods and fields provides an understanding of the problem and some important language concepts.  

## Before ES2022

### The Problem, A Weak Pillar

Languages that implement the Object-Oriented Programming paradigm have to implement the four pillars of Object-Oriented Programming.  Those pillars being: abstraction, inheritance, polymorphism, and encapsulation.  While JavaScript met the requirements of implementing abstraction, inheritance, and polymorphism, there was debate if JavaScript satisfied the requirement of implementing encapsulation.  

JavaScript lacked a native way to make data not public.  This meant that there was no way to enforce encapsulation out of the box.  This can lead to the conclusion that JavaScript was not a language that can use the OOP programming paradigm, but smart people found a way to implement encapsulation by using some JavaScript features: closure, weak maps, and symbols.  

### A Solution using Closure

#### What is Closure?

A closure is the idea of saving scope and the function declarations or data block to memory to keep the scope alive after a function or block would have been normally consumed and discarded.  

```JavaScript

let saveScope = function(){

    let number = 5;

    return {

        getNumber: function(){

            return number;
        }
    }
}

let useScope = saveScope();
/* saveScope will return the block and its scope:
    {

        getNumber: function(){

            return number;
        }
    }
*/

useScope.number // undefined, 
//because there is no number parameter inside the block
useScope.getNumber() // 5, 
//because the variables that were in scope were 
//saved along with the function declaration

```

Normally after a function is used, it is popped off the execution stack and the variables are deleted to create free space in memory (in memory means in the computer's RAM).  

#### Private Variables Using Closure

The idea is to create function expression that returns an object that contains setter and getter functions.  Those functions will create a closure over a variable, meaning the variable will only be accessible through those functions.

```JavaScript

const numberCreator = function(initalValue){

  let aNumber = initalValue0;

  return{
      getNumber: function(){

          return aNumber;
      },

      setNumber: function(number){

          aNumber = number

      }
  }

}

const aNumber = numberCreator(0)

aNumber.getNumber() // 0
aNumber.setNumber(10)
aNumber.getNumber() // 10
```

### A Solution Using Weak Maps

#### What Are Weak Maps

A weak map is a data structure similar to a JavaScript object.  It differs from a normal object in that the key must be an object.

Weak Maps have the following built-in methods:

- weakMap.set(key, value)

  - Sets the key/value pair, the key must be an object.  For this use case the key will be an instance of the class.  

- weakMap.get(key)

  - Returns the value that corresponds with the key that was passed in.  

- weakMap.has(key)

  - Checks if the key exists in the weak map.  Returns a Boolean value.  

- weakMap.delete(key)

  - Deletes the key/value pair associated withe the key passed in.  

#### Private Variables Using Weak Maps

The idea is to create a weak map that will hold an instance of the class as the key and an object consisting of private properties and methods as the value.  

An IIFE is created to create the weak map and return the class declaration.  

```JavaScript
let Car = (function(){

  let privateProperties = new WeakMap();

  return class Car{

      constructor(make, model, year){

          // Public Variables
          this.make= make;
          this.model = model;

          // Private
          privateProperties.set(this, 
            {
                year: year,
                message: function(name){

                    return `Hello ${name}`
                }
            })

      }

      get getYear(){

        // The get built-in method of a WeakMap requires a key, 
        // which must be an object, to return the value associated with it.  
        // Passing in this keyword to the get method queries the WeakMap 
        //for the object that has the current instance of the class as a key
        // It returns the object with the private methods and fields.   
        return privateProperties.get(this).year;

      }

      message(name) {

        return privateProperties.get(this).message(name)

      }

  }

})()

let myCar = new Car("Ford", "Mercury", 1998);

myCar.make // Ford
myCar.getYear // 1998
myCar.message("Henry") // Hello Henry
```

#### Two Ways to implement

Because the solution uses a weak map that is outside of the class there are two options to manage the private data.  The first is to create a new weak map per class instance, this is the option example above uses.  The other is to create a global weak map and add to it as new instances of a class are created, as shown below.  

The second option makes better use of the abilities of the weak map.  Only one weak map is created with multiple object/value pairs, which is how weak maps are meant to be used.  unfortunately, that weak map must be in a scope wide enough that it can be used by all instances of the class, which may very well mean it has to be a global data structure.  

```JavaScript

  // This Weak Map will hold the private properties f
  // or all the instances of the Car class
  let carPrivateProperties = new WeakMap();

  class Car{

      constructor(make, model, year){

          // Public Variables
          this.make= make;
          this.model = model;

          // Private
          carPrivateProperties.set(this, 
            {
                year: year,
                message: function(name){

                    return `Hello ${name}`
                }
            })

      }

      get getYear(){
 
        return carPrivateProperties.get(this).year;

      }

      message(name) {

        return carPrivateProperties.get(this).message(name)

      }

  }

let myCar = new Car("Ford", "Mercury", 1998);
let myCar2 = new Car("Toyota", "Camry", 2015);

myCar.make // Ford
myCar.getYear // 1998
myCar.message("Henry") // Hello Henry

myCar2.make // Toyota
myCar2.getYear // 2015
myCar2.message("Kiichiro") // Hello Kiichiro 
```

### A Solution using Symbols

#### What are Symbols?

A symbol is a primitive type that is created using the Symbol object's constructor function, which returns a symbol.  All symbols are guaranteed to be unique and do not appear as properties of an object.

Object.getOwnPropertySymbols() can be used to see an object's symbols, which means that private methods and fields are not totally encapsulated. 

#### Private Variables Using Symbols

```JavaScript

let Car = (function(){

  let yearKey = Symbol();
  let messageKey = Symbol();

  return class Car {

    constructor(make, model, year){
      

          // Public Variables
          this.make= make;
          this.model = model;

          // this.yearKey = year will not work
          this[yearKey] = year;
          this[messageKey] = function(name){

                    return `Hello ${name}`
                }

      }

      get getYear(){

        return `year ${this[yearKey]}`;

      }

      message(name) {

        return this[messageKey](name);

      }

  }
})()

let myCar = new Car("Ford", "Mercury", 1998);
let myCar2 = new Car("Toyota", "Camry", 2015);

myCar.make // Ford
myCar.getYear // 1998
myCar.message("Henry") // Hello Henry

myCar2.make // Toyota
myCar2.getYear // 2015
myCar2.message("Kiichiro") // Hello Kiichiro 

```

This classes can be extended as normal.

```JavaScript
let Animal = (function(){

  let idKey = Symbol();

  return class Animal {

    constructor(id){
      
          this[idKey] = id;
          

      }

      get getId(){

        return `I am ${this[idKey]}`;

      }

  }
})()

const animal = new Animal(12345);

animal.getId; // I am 12345

class Duck extends Animal {

    constructor(id){

        super(id)

        // Public Variables
        this.species = "Duck";

        this.noise = "Quack";
    }

}

let jerry = new Duck(23456);

jerry.getId; // I am 23456
jerry.species; // Duck
jerry.noise; // Quack
```

## As of ES2022

ES2022 introduced a way to create fields and methods natively and conveniently using a hash (this sign #) placed before the name of the target field or method.

This better because it is simpler and less clever (does not seem like a hack).  Private methods and fields there are also no way to 

This feature is accepted by most of the most popular browsers.  According to data from [Can I Use](https://caniuse.com/?search=private) 88%+ of users use a browser that has implemented private fields and methods using the hash syntax.  

Finally, here is an example of private fields and methods using the hash syntax.  

```JavaScript
class Animal {

  // private methods and fields have to be declared at the top
  #id;

  // Private fields can also be assigned values when they are declared
  //#id = 123243;

  constructor(id){
    
    this.#id = id;

  }

  get getId(){

    return `I am ${this.#id}`;

  }
}

let animal = new Animal(12345);

animal.getId; // I am 12345

class Duck extends Animal {

  constructor(id){

      super(id)

      // Public Variables
      this.species = "Duck";
      this.noise = "Quack";
  }

}
  
let jerry = new Duck(23456);

jerry.getId; // I am 23456
jerry.species; // Duck
jerry.noise; // Quack
```

## Conclusion

Using hash notation is the best option when creating private methods and fields.  This being said, the older ways of creating private methods and fields are worth exploring because they provide an opportunity to learn key concepts of JavaScript.
