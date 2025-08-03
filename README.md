# On the way to functional programming

2025-07-30

How to make functional programming when you have your feet tied to the imperative and mutable code?

## Red thread project

- Feature :
**As** an event organizer, **I want to** search for registered attendees by first name, **in order to** be able to create the list of attendees actually present at the event.

## List test case

0, 1, many…

## Acceptance criteria

- The finder should return :

  - an empty result when no attendee first name matches the query string.
  - one result when only one attendee first name matches the query string.
  - all the results when many attendees first names match the query string.

## Setup of JS project

[Setup](setup-javascript-project.md)

## Unit test setup

```js
describe("The attendees finder should return", () => {
  let attendees = [];

  beforeEach(() => {
      attendees = [
        {
          firstName: "Marc",
        },
        {
          firstName: "Christelle",
        },
        {
          firstName: "Christophe",
        },
      ];
  });
```

## Unit test case

```js
    test("an empty result when no attendee first name matches the query string", () => {
    const result = findByInfixOfFirstName("Paul", attendees);

    expect(result).toStrictEqual([]);
  });
  
  test("one result when only one attendee first name matches the query string", () => {
    const result = findByInfixOfFirstName("Marc", attendees);

    expect(result).toStrictEqual([{ firstName : "Marc"}]);
  });

  test("all the results when many attendees first names match the query string", () => {
    const result = findByInfixOfFirstName("Chri", attendees);

    expect(result).toStrictEqual([{ firstName : "Christelle"}, { firstName: "Christophe"}]);
  });
```

## First implementation

```js
export function findByInfixOfFirstName(query, attendees) {
    const result = [];

    let attendee;
    while (attendee = attendees.shift()) {
        if (attendee.firstName.includes(query)) {
            result.push(attendee);
        }
    }
    return result;
}
```

## Unit test setup - Unique list for all tests

```js
  let attendees = [];

  beforeAll(() => {
      attendees = [
        {
          firstName: "Marc",
        },
        {
          firstName: "Christelle",
        },
        {
          firstName: "Christophe",
        },
      ];
  });
```

## First imperative implementation with side effect

```js
export function findByInfixOfFirstName(attendees, query) {
    const result = [];

    let attendee;
    while (attendee = attendees.shift()) {
        if (attendee.firstName.includes(query)) {
            result.push(attendee);
        }
    }
    return result;
}
```

## Fix side effects

```js
export function findByInfixOfFirstName(attendees, query) {
    const result = [];

    for (const attendee of attendees) {
        if (attendee.firstName.includes(query)) {
            result.push(attendee);
        }
    }
    return result;
}
```

## Put side effects at the periphery

![Referential transparency](referential-transparency.png)

## From imperative to procedural paradigm

> Extract function `matches`

```js
export function findByInfixOfFirstName(query, attendees) {
    const result = [];

    for (const attendee of attendees) {
        if (matches(attendee, query)) {
            result.push(attendee);
        }
    }
    return result;
}

function matches(attendee, query) {
    return attendee.firstName.includes(query);
}
```

## Suppress second level of indentation

```js
export function findByInfixOfFirstName(query, attendees) {
    const result = [];

    for (const attendee of attendees) {
        addIfMatches(attendee, query, result);
    }
    return result;
}

function addIfMatches(attendee, query, result) {
    if (matches(attendee, query)) {
        result.push(attendee);
    }
}

function matches(attendee, query) {
    return attendee.firstName.includes(query);
}
```

## Looking for side effects

If a method signature indicates a return nothing and the method is used, there is a side effect.

```js
function addIfMatches(attendee, query, result) {
    if (matches(attendee, query)) {
        result.push(attendee);
    }
}
```

## Function as value - First class citizen

Defines and used a constant named `predicate` that is a function. The constant can be used as parameter.

```js
export function findByInfixOfFirstName(query, attendees) {
    const result = [];

    const predicate = (attendees, query) => matches(attendees, query);

    for (const attendee of attendees) {
        addIfMatches(predicate, attendee, query, result);
    }
    return result;
}

function addIfMatches(predicate, attendee, query, result) {
    if (predicate(attendee, query)) {
        result.push(attendee);
    }
}

function matches(attendee, query) {
    return attendee.firstName.includes(query);
}
```

## Closure - Avoid to pass result to addIf

The `append` function uses the `result` parameter in a closed manner. This is called a closure.

```js
export function findByInfixOfFirstName(query, attendees) {
    const result = [];

    const predicate = (attendees, query) => matches(attendees, query);
    const append = attendee => result.push(attendee);

    for (const attendee of attendees) {
        addIfMatches(predicate, attendee, query, append);
    }
    return result;
}

function addIfMatches(predicate, attendee, query, append) {
    if (predicate(attendee, query)) {
        append(attendee);
    }
}

function matches(attendee, query) {
    return attendee.firstName.includes(query);
}
```

## Higher order function - To not be silly, let others do

```js
export function findByInfixOfFirstName(query, attendees) {
    const result = [];

    const predicate = (attendees, query) => matches(attendees, query);
    const append = attendee => result.push(attendee);

    for (const attendee of attendees) {
        addIfMatches(predicate, attendee, query, append)(attendee);
    }
    return result;
}

function addIfMatches(predicate, attendee, query, append) {
    if (predicate(attendee, query)) {
        return append;
    } else {
        return () => {};
    }
}

function matches(attendee, query) {
    return attendee.firstName.includes(query);
}
```

## HoF - Change matches and remove `query` parameter of `addIfMatches` function

The `matches` function that has two parameters is transformed into a one-parameter function that returns another function with one parameter.
This is called **currying**.

The `predicate` constancy is initialized with a function reference.

The `query` parameter can be remove of `addIfMatches` function.

```js
export function findByInfixOfFirstName(query, attendees) {
    const result = [];

    const predicate = matches(query);
    const append = attendee => result.push(attendee);

    for (const attendee of attendees) {
        addIfMatches(predicate, attendee, append)(attendee);
    }
    return result;
}

function addIfMatches(predicate, attendee, append) {
    if (predicate(attendee)) {
        return append;
    } else {
        return () => {};
    }
}

function matches(query) {
    return attendee => attendee.firstName.includes(query);
}
```

## Replace `append` by `concat` - Be honest

```js
export function findByInfixOfFirstName(query, attendees) {
    let result = [];

    const predicate = matches(query);
    const concat = (attendee, attendees) => attendees.concat(attendee);

    for (const attendee of attendees) {
        result = addIfMatches(predicate, attendee, concat)(attendee, result);
    }
    return result;
}

function addIfMatches(predicate, attendee, concat) {
    if (predicate(attendee)) {
        return concat;
    } else {
        return (_attendee, attendees)=> attendees;
    }
}

function matches(query) {
    return attendee => attendee.firstName.includes(query);
}
```

## Currying concat

```js
export function findByInfixOfFirstName(query, attendees) {
    let result = [];

    const predicate = matches(query);
    const concat = attendee => attendees => attendees.concat(attendee);

    for (const attendee of attendees) {
        result = addIfMatches(predicate, attendee, concat)(attendee)(result);
    }
    return result;
}

function addIfMatches(predicate, attendee, concat) {
    if (predicate(attendee)) {
        return concat;
    } else {
        return _attendee => attendees=> attendees;
    }
}

function matches(query) {
    return attendee => attendee.firstName.includes(query);
}
```

## Partial application

The call of `concat` function is with only one parameter `attendee`.

```js
export function findByInfixOfFirstName(query, attendees) {
    let result = [];

    const predicate = matches(query);
    const concat = attendee => attendees => attendees.concat(attendee);

    for (const attendee of attendees) {
        result = addIfMatches(predicate, attendee, concat)(result);
    }
    return result;
}

function addIfMatches(predicate, attendee, concat) {
    if (predicate(attendee)) {
        return concat(attendee);
    } else {
        return attendees => attendees;
    }
}

function matches(query) {
    return attendee => attendee.firstName.includes(query);
}
```

## Extract `filter` function

```js
export function findByInfixOfFirstName(query, attendees) {
    
    const predicate = matches(query);
    
    return filter(predicate, attendees);
}

function filter(predicate, attendees) {
    const concat = attendee => attendees => attendees.concat(attendee);
    let result = [];
    for (const attendee of attendees) {
        result = addIfMatches(predicate, attendee, concat)(result);
    }
    return result;
}

function addIfMatches(predicate, attendee, concat) {
    if (predicate(attendee)) {
        return concat(attendee);
    } else {
        return attendees    => attendees;
    }
}

function matches(query) {
    return attendee => attendee.firstName.includes(query);
}
```

## Simplify `filter` function

```js
export function findByInfixOfFirstName(query, attendees) {
    
    const predicate = matches(query);
    
    return filter(predicate, attendees);
}

function filter(predicate, attendees) {
    return attendees.filter(predicate, attendees)
}


function matches(query) {
    return attendee => attendee.firstName.includes(query);
}
```

## Final version

```js
export function findByInfixOfFirstName(query, attendees) {
    return filter(matches(query), attendees);
}

function filter(predicate, attendees) {
    return attendees.filter(predicate, attendees)
}

function matches(query) {
    return attendee => attendee.firstName.includes(query);
}
```

## Setup haskell project

[Setup](./setup-haskell-project.md)

## Unit test case in haskell

```haskell
import Test.Hspec
import AttendeesFinder(findByInfixOfFirstName, Attendee(firstName, Attendee))

main = hspec $ do
    let attendees = [Attendee { firstName = "Marc" }, 
                     Attendee { firstName = "Christelle" },
                     Attendee { firstName = "Christophe" }]

    describe "The attendees finder should return" $ do
        it "an empty result when no attendee first name matches the query string" $ do
            findByInfixOfFirstName "Paul" attendees `shouldBe` []

        it "one result when only one attendee first name matches the query string" $ do
            findByInfixOfFirstName "Marc" attendees `shouldBe` [Attendee {firstName = "Marc"}]

        it "all the results when many attendees first names match the query string" $ do
            findByInfixOfFirstName "Chri" attendees `shouldBe` [Attendee {firstName = "Christelle"}, Attendee {firstName = "Christophe"}]
```

## First module `AttendeesFinder`

```haskell
module AttendeesFinder(
    findByInfixOfFirstName, 
    Attendee(firstName, Attendee)
) where


data Attendee = Attendee {
     firstName :: String
} deriving (Show, Eq)


findByInfixOfFirstName :: String -> [Attendee] -> [Attendee]
findByInfixOfFirstName = undefined
```

## First implementation in haskell

```haskell
module AttendeesFinder(
    findByInfixOfFirstName, 
    Attendee(firstName, Attendee)
) where

import Data.List(isInfixOf)

data Attendee = Attendee {
     firstName :: String
} deriving (Show, Eq)


findByInfixOfFirstName :: String -> [Attendee] -> [Attendee]
findByInfixOfFirstName query attendees = filter (matches query) attendees
    where matches :: String -> Attendee -> Bool 
          matches query attendee = query `isInfixOf` (firstName attendee)
```

## Simplify by using function composition

```haskell
module AttendeesFinder(
    findByInfixOfFirstName, 
    Attendee(firstName, Attendee)
) where

import Data.List(isInfixOf)

data Attendee = Attendee {
     firstName :: String
} deriving (Show, Eq)


findByInfixOfFirstName :: String -> [Attendee] -> [Attendee]
findByInfixOfFirstName = filter . matches
    where matches :: String -> Attendee -> Bool 
          matches query attendee = query `isInfixOf` (firstName attendee)
```
