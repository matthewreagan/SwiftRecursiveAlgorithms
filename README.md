<p align="center"><img src="https://developer.apple.com/assets/elements/icons/swift/swift-64x64.png"/></p>

# Swift Graph Traversal & Recursive Algorithms

This is a list of solutions to common [recursive](https://en.wikipedia.org/wiki/Recursion#In_computer_science) or [graph traversal](https://en.wikipedia.org/wiki/Graph_traversal) problems. [Iterative](https://www.codeproject.com/Articles/21194/Iterative-vs-Recursive-Approaches) solutions are sometimes also shown (though any problem that can be solved recursively can also be solved iteratively). The purpose is to assist in understanding how to approach and solve these types of problems (the solutions should not be memorized or simply copy-pasted).

The algorithms here are presented in **Question-Answer** format, and in most cases there are a multitude of possible solutions. The solutions here are _not necessarily the most efficient_, but rather those that provide the clearest insight into how the solution is actually reached. There are also often problem-specific assumptions made that would not be safe to make in more generalized problem domains (assertions and error handling are typically omitted).

For a more exhaustive list of specific algorithm examples see: [Swift Algorithm Club](https://github.com/raywenderlich/swift-algorithm-club).

**Final disclaimer**: this is a work in progress. I'm mostly writing this up for my own amusement. Suggestions / contributions are welcome.

#### Problems & Solutions
- [Fibonacci Sequence](#fibonacci)
- [String Permutations](#string-permutations)
- [N-Queens Problem](#n-queens-problem)
- [N-Parentheses Pairs](#n-parentheses-pairs)
- [Recursive atoi](#recursive-atoi)
- [Paint Fill Algorithm](#paint-fill-algorithm)
- [Change For A Dollar](#change-for-a-dollar)
- [Alphabetic Phone Number](#alphabetic-phone-number)
- [Boggle](#boggle)

---

## Fibonacci
### **Problem**: Write a function to generate the Nth Fibonacci number.

*Discussion*: This is a great example of how to approach writing a recursive function, since the solution is so succinct. The Nth [Fibonacci number](https://en.wikipedia.org/wiki/Fibonacci_number) is the sum of the previous two Fibonacci numbers, and in practice the sequence looks like this:

![fibonacci](https://wikimedia.org/api/rest_v1/media/math/render/svg/ff1558cc1c547dba59ce03cf352c1662b87d10f1)

So to write a recursive function we can start with the base case, `f(1)` or `f(2)`, for which we simply hardcode the return values of 1. From there we can build upon that by recursing to find `f(n)`, by simply returning the sum of `f(n-2)` and `f(n-1)`.

*Notes*: For this example, N <= 0 is considered invalid, though we omit the related guard/assert for the sake of brevity.

*Recursive solution*
```swift
func fib(_ n: Int) -> Int {
    // Base case, when N == 1 or N == 2:
    if n <= 2 { return 1 }
    // For any other N, we return the sum of the previous two numbers:
    return fib(n-2) + fib(n-1)
}
```

## String Permutations
### **Problem**: Find all possible permutations of a string.

*Discussion*: As is frequently the case, one of the most important aspects of this simple problem is recognizing patterns in how the subproblem is structured. We can start with the simplest case, string 'ab', in which we have 2 possible permutations: 'ab' and 'ba'. Note that we could think of this as taking 'a', and placing it in each possible position around the rest of the string, 'b'.

If we then try 'xab', we see that the permutations are 'xab', 'axb', 'abx', 'xba', 'bxa', 'bax'. Essentially we're taking the preceeding character ('x'), computing the permutations of the remaining substring 'ab', and then simply inserting the character into each possible position for the substring. From this we can formulate a recursive solution.

**This is a good example of a general approach for writing a recursive algorithm: start by solving the simplest scenario or base case first, and then build off of that.**

*Notes*: The number of possible permutations is N! ([factorial](https://en.wikipedia.org/wiki/Factorial)), where N is the number of characters in the string.

*Caveats*: This solution does not take into consideration the uniqueness of each letter in the string, e.g. 'aa' will output with two equivalent permutations ('aa', 'aa').

*Recursive solution*
```swift
func permu(_ input: String) -> [String] {
    assert(!input.isEmpty)
    
    // Handle base case, a single letter
    if input.count == 1 {
        return [input]
    } else {
        // Grab the first character, calculate
        // the permutations for the remainder
        // of the string, and then insert the
        // first character into each position.
        // For 'ab', this is taking 'a', calculating
        // permutations for 'b' (which is just ['b']),
        // and then inserting 'a' into each position
        // around it: 'ab', 'ba'.
        
        var output: [String] = []
        let firstCharacter = input.first!
        let remainder = String(input.dropFirst())
        
        // Here is where we recurse, for
        // progressively smaller substrings:
        let permus = permu(remainder)
        
        for p in permus {
            for pos in 0...(p.count) {
                var newStr = p
                newStr.insert(firstCharacter, at:newStr.index(newStr.startIndex, offsetBy: pos))
                output.append(newStr)
            }
        }
        
        return output
    }
}

let permutations = permu("abc")
print("Total: \(permutations.count)")
```
*Output*: 
```
["abc", "bac", "bca", "acb", "cab", "cba"]
Total: 6
```
## N Queens Problem
### **Problem**: Find the number of ways 8 queens can be placed on a chess board such that no queen attacks another

*Discussion*: This is the standard variant of the [N Queens Problem](https://en.wikipedia.org/wiki/Eight_queens_puzzle), and amounts to finding all of the possible positions 8 queens can be placed such that no queen shares the same column, row, or diagonal as another. As with other problems, solving this is much easier if you take a moment to identify assumptions or shortcuts that can be made safely within the problem space. Approaching this with the mindset that any queen could occupy any arbitrary position in the board can become overwhelming, and is unnecessary given the rules we're presented.

Because a chess board is an 8x8 grid, and we know that only 1 queen can ever be placed in a single column, we can simplify both our data structures and code by working under these assumptions. Rather than store the arbitrary position of a given queen (as an `X,Y` pair for example), we can simply use a data structure that represents a particular row for a given column (an `Array<Int>`, with 8 values).

*Notes*: This solution is an example of a [Backtracking algorithm](https://en.wikipedia.org/wiki/Backtracking). Many of the other problems here are also solved similarly: by using the recursive function stack as a means to explore a tree of possibilities, and then _backtracking_ once we reach an invalid combination or deadend. As with all backtracking algorithms, we take advantage of our ability to compute a [_partial candidate solution_](https://www.techopedia.com/definition/17837/backtracking). This simply means that as we explore each possible combination, if we reach a point where we know we can't place the Nth queen, then any subsequent queens in that tree of possibilities can be ignored (since the entire subtree is invalid).

*Recursive solution*
```swift
var board: [Int] = Array(repeating: 0, count: 8)
var totalPermutations = 0

func canPlace(at pos: (x: Int, y: Int)) -> Bool {
	
    // We know when we're checking the placement of
    // X,Y we've already successfully placed queens
    // up to the current column, so we only need to
    // check those previously-placed queens
    
    let thisQueenCol = pos.x
    let thisQueenRow = pos.y
    
    if thisQueenCol == 0 {
        // The first queen (in first column) can always be placed in any row
        return true
    }
    
    // We check the previously-placed queens up to pos.X
    for (queenCol, queenRow) in board.prefix(thisQueenCol).enumerated() {
        // If this previously-placed queen is in the same row as our
        // current proposed queen (its Y), the placement is invalid
        if queenRow == thisQueenRow {
            return false
        }
        
        // If this previously-placed queen is in the same diagonal
        // as our current proposed queen, also invalid
        let dx = queenCol - thisQueenCol
        let dy = queenRow - thisQueenRow
        
        if dx.magnitude == dy.magnitude {
            return false
        }
    }
    
    return true
}

// We're going to iterate over every possible row (Y)
// in each column (X) that a queen could be placed,
// recursing down through the tree of possibilities
// for each subsequent column

func placeQueen(at col: Int) {
	
    // Handle base case. If we're at column 8, we've placed
    // all 8 queens successfully.
	
    if col == 8 {
        totalPermutations += 1
        printBoard(board)
    } else {
		
		// For each Y in this column, attempt to place
		// the queen (by checking the previously-placed)
		// queens in earlier columns. Because our recursive
		// algorithm explores only a single board layout at
		// a time, and in order, we can simply reuse our board
		// variable without worrying about clearing it etc.
		
        for y in 0..<8 {
            if canPlace(at: (x: col, y: y)) {
                board[col] = y
                placeQueen(at: col + 1)
            }
        }
    }
}

placeQueen(at: 0)
print("Total permutations: \(totalPermutations)")
```

*Example output (only 1 printed board shown)*:
```
__Q_____
_____Q__
___Q____
_Q______
_______Q
____Q___
______Q_
Q_______

Total permutations: 92
```

## N Parentheses Pairs
### **Problem**: Find all valid combinations of N pairs of parentheses.

*Discussion*: This is another problem which rewards us for taking a moment in the beginning to understand what assumptions can be made. When given N pairs of parentheses, we need to find all valid combinations. So for example, when N is 2 the valid possibilities would be `()()` and `(())`. From these basic rules we can notice a few key takeaways:

1. A valid combo will be a string of length `N * 2`
2. We'll need a way to test _partial candidate solutions_ and valid combos
3. We have a limited number of open and closed parentheses. Depending on which we place early on, we may run out one or the other, limiting our choices

From these rules we can see that a recursive backtracking algorithm will allow us to find all possbilities. We iterate over each position in the string, trying **both** an open and closed parenthesis if possible. (If we've used up all of one, we obviously can only try the other.) As we're testing each possible placement in the string, we can check the validity.

How do we check for a valid combo? One straightforward way is to ascribe a positive and negative value to open `(` and closed `)` parentheses, respectively. We scan our string, adding 1 for each `(` and -1 for each `)`. If we ever find that our sum has gone below zero, we know we have an invalid combo. We can test this out in practice by noticing what the sums would be for various valid and invalid combinations tested in this way:

```
((())) = +1 +1 +1 -1 -1 -1 = 0
()()() = +1 -1 +1 -1 +1 -1 = 0
())(() = +1 -1 -1    _Our sum is < 0, combo is invalid_
)))((( = -1 _Our sum is < 0, combo is invalid_
```

Now we can write up a fairly simple recursive algorithm which tests each possible `)` or `(` in each position. Note that each time we recurse for a new branch of possibilities, we keep track of the available number of `)` and `(` to make sure we don't overuse one or the other.

*Recursive solution*
```swift
let numPairs = 3
let strLength = numPairs * 2
var str = String(repeating: " ", count: strLength)

func isValid<T: StringProtocol>(_ str: T) -> Bool {
    var sum = 0
    for c in str {
        if c == "(" { sum += 1 }
        else if c == ")" { sum -= 1 }
        if sum < 0 { return false }
    }
    return true
}

func recurse(_ index: Int, available: (open: Int, closed: Int)) {
    
    // Base case, if we've reached the end of the string
    // (or we've run out of both open / closed parentheses,
    // which should always occur simultaneously)
    
    if index >= strLength {
        if isValid(str) {
	    // Success
            print("\(str)")
        }
    } else {
        
        // Here is where we recurse. We try both an open
        // and closed parenthesis in this position (if available).
        
        let open = available.open
        let closed = available.closed
        let charIndex = str.index(str.startIndex, offsetBy: index)
	
	// Note that in both of the checks below, we test only
	// the prefix portion of our partial candidate solution
	// up to the current index, since we haven't yet placed
	// any parentheses beyond that (though because we're
	// reusing a single string, the actual string data may
	// have other '(' or ')' characters in it from previous
	// attempts)
        
        if open > 0 {
            str.remove(at: charIndex)
            str.insert("(", at: charIndex)
            
            if isValid(str.prefix(index)) {
                recurse(index + 1, available: (open: open - 1, closed: closed))
            }
        }
        
        if closed > 0 {
            str.remove(at: charIndex)
            str.insert(")", at: charIndex)
            
            if isValid(str.prefix(index)) {
                recurse(index + 1, available: (open: open, closed: closed - 1))
            }
        }
    }
}

recurse(0, available: (open: numPairs, closed: numPairs))
```

*Example output (N = 3)*:
```
((()))
(()())
(())()
()(())
()()()
```
## Recursive atoi
### **Problem**: Write a recurisve implementation of atoi()

*Discussion*: `atoi()` takes a string and converts it to an integer. As with our previous problems, it's helpful to begin by identifying the key subproblem, or identifying the base case, and building from there. We can see that if we have a single character string `"5"`, conversion is quite simple: we get the numeric character value (in whatever way is approrpriate for the implementation) and convert it to an equivalent `Int`, so in this example we take the ASCII value of `"5"` which is 53, and subtract 48 (since the range of ASCII numeric values we're interested in is 48-57, inclusive).

If we have a 2 character string `"25"`, we see where our recursion comes in. We need to recurse over each character in the string, and continually add to our resulting `Int` the numeric value of the character, multiplied by `10 ^ P` where P is the index in the string. So our function should ultimately yield:

`2 * (10 ^ 1) + 5 * (10 ^ 0) = 25`

*Notes*: The solution below omits error checking for unexpected / non-numeric characters.

*Recursive solution*
```swift
func myAtoi(_ str: String,
            index: Int = 0,
            sum: Int = 0) -> Int {
    
    // Grab the character at `index` and convert it to
    // an integer based on the ASCII value. (Note that
    // we don't do any extensive error checking here for
    // the sake of simplicity, a real implementation would
    // need to be more robust.)
    
    let place = str.count - index - 1
    let c = str[str.index(str.startIndex, offsetBy: index)]
    let asciiValue = Int(c.unicodeScalars.first!.value)
    guard asciiValue >= 48 && asciiValue <= 57 else { return 0 }
    let intValue = (asciiValue - 48) * Int(pow(10.0, Double(place)))
    
    // Base case, last digit:
    if place == 0 {
        return intValue
    } else {
        // Here we recurse, adding this digit's value to our total
        // sum and then continuing for the next character as needed.
        
        return sum + intValue + myAtoi(str, index: index + 1, sum: sum)
    }
}

let x = myAtoi("1042")
```

## Paint Fill Algorithm
### **Problem**: Implement a paint bucket / fill algorithm.

*Discussion*: This can be thought of as a basic graph / tree traversal. We can think of the pixel the user clicks on as the starting parent node, and each surrounding node as a child. We continue to explore all of the children's children and set them to the new color or value as necessary.

*Notes*: For the purposes of this demo code we assume we're filling any ' ' spaces (pixels) in our character grid with '•'. This solution allows diagonal movements, but that could be changed easily with an additional check of our x/y neighbor iteration. The `pixel()`, `setPixel()`, and `printPixels()` functions have been omitted since they're fairly basic and implementation-specific.

*Recursive solution*
```swift
var pixels = ["XXXXXXXXXXXX",
              "XXXX    XX X",
              "XX     XXX X",
              "X XX   XX XX",
              "X XXX  X  XX",
              "X XX    XX X",
              "XXXX       X"]
let gridWidth = 12; let gridHeight = 7
struct Pos { let x,y: Int }

func notOutOfBounds(_ p: Pos) -> Bool {
    return p.x >= 0 &&
        p.x < gridWidth &&
        p.y >= 0 &&
        p.y < gridHeight
}

func paintFill(at pos: Pos) {
    
    // Check the pixel at this position
    if pixel(at: pos) == " " {
        
        // Fill this pixel if needed
        setPixel(at: pos, to: "•")
        
        // Iterate over all neighbors
        for y in -1...1 {
            for x in -1...1 {
                
                // Skip (0,0), the current pixel
                if (x != 0 || y != 0) {
                    
                    let newPos = Pos(x: pos.x + x, y: pos.y + y)
                    if notOutOfBounds(newPos) {
                    
                        // Recurse to fill all neighbor pixels
                        paintFill(at: newPos)
                    }
                }
            }
        }
    }
}

paintFill(at: Pos(x: 4, y: 2))
pixels.printPixels()
```

*Iterative solution*
```swift
func paintFill(at startingPos: Pos) {
    
    // Begin our list of pixels to check with the starting position
	// NOTE: This function is inefficient because it re-checks pixels
	// we've already visited. This could be easily avoided by keeping
	// track of visited pixels with a 'closed' list
	
    var pixelsToVisit: [Pos] = []
    pixelsToVisit.append(startingPos)
    
    while !pixelsToVisit.isEmpty {
        
        // Get a pixel to check, remove it from our ToDo list
        let pos = pixelsToVisit.removeFirst()
        
        // Check the pixel at this position
        if pixel(at: pos) == " " {
            
            // Fill this pixel if needed
            setPixel(at: pos, to: "•")
            
            // Iterate over all neighbors
            for y in -1...1 {
                for x in -1...1 {
                    
                    // Skip (0,0), the current pixel
                    if (x != 0 || y != 0) {
                        
                        let newPos = Pos(x: pos.x + x, y: pos.y + y)
                        if notOutOfBounds(newPos) {
                            
                            // Rather than recurse, we add the neighbor
                            // pixels to our open list
                            pixelsToVisit.append(newPos)
                        }
                    }
                }
            }
        }
    }
}
```

*Output*:
```
XXXXXXXXXXXX
XXXX••••XX•X
XX•••••XXX•X
X•XX•••XX•XX
X•XXX••X••XX
X•XX••••XX•X
XXXX•••••••X
```

## Change For A Dollar
### **Problem**: Calculate all of the possible combinations of change which can total $N using ∞ quarters, dimes, nickels, and pennies

*Discussion*: This problem can be understood slightly easier by reading through the iterative solution below which hardcodes each of the individual coin denominations. While not the best approach, it helps illustrate what we're actually doing: starting with the largest coin, we branch on each number of coins that could fit into our total, and then for the remaining amount we compute the number of next-largest coins that would fit, further branching on each of those possibilities, and so on. The recursive solution below is more flexible since it can use any arbitrary combination of coins.

*Iterative example*
```swift
let targetTotal = 100 // $1
var totalPermutations = 0

func countChange() {
    for quarters in 0...(targetTotal / 25) {
        let leftoverAfterQuarters = targetTotal - (quarters * 25)
        for dimes in 0...(leftoverAfterQuarters / 10) {
            let leftoverAfterDimes = leftoverAfterQuarters - (dimes * 10)
            for nickels in 0...(leftoverAfterDimes / 5) {
                let leftoverAfterNickels = leftoverAfterDimes - (nickels * 5)
                for pennies in 0...leftoverAfterNickels {
                    if quarters * 25 + dimes * 10 + nickels * 5 + pennies == targetTotal {
                        totalPermutations += 1
                        print("Q: \(quarters) D: \(dimes) N: \(nickels) P: \(pennies)")
                    }
                }
            }
        }
    }
}
countChange()
print ("Total permutations: \(totalPermutations)")
```

*Recursive solution*
```swift
let targetTotal = 100 // $1
var totalPermutations = 0
let denominations = [25, 10, 5, 1]

func countChangeRecursive(_ amt: Int, denomination: Int, previousTotal: Int) {
    for coins in 0...(amt / denomination) {        
        guard previousTotal != targetTotal else { continue }
        let thisTotal = (coins * denomination) + previousTotal
        if thisTotal == targetTotal { totalPermutations += 1 }
        
        let leftover = targetTotal - thisTotal
        let nextDenomIndex = denominations.index(of: denomination)! + 1
        if nextDenomIndex < denominations.count {
            let nextDenomination = denominations[nextDenomIndex]
            countChangeRecursive(leftover,
                                 denomination: nextDenomination,
                                 previousTotal: thisTotal)
        }
    }
}

countChangeRecursive(targetTotal, denomination: denominations[0], previousTotal: 0)
print ("Total permutations: \(totalPermutations)")
```

## Alphabetic Phone Number
### **Question**: Find all letter combinations for a 7 digit phone number

*Discussion*: Given a standard 7-digit phone number, calculate all the possible word combinations that could be formed using the standard letters on a telephone numeric pad. 

*Notes*: The standard telephone keypad has no numbers for digits 1 and 0, and some of the numbers can be represented by 4 (not 3) possible letters. 

*Caveats*: The solution below calculates all possible letter / word combinations, not just ones that appear in the English dictionary.

*Recursive solution*
```swift
let number = "9653669"
var word = "AAAAAAA"
let numbersToLetters: [String: String] =
    ["1" : "", "2" : "ABC", "3" : "DEF",
     "4" : "GHI", "5" : "JKL", "6" : "MNO",
     "7" : "PQRS", "8" : "TUV", "9" : "WXYZ", "0" : ""]

func calculateWords(_ phoneNumber: String, numeral: Int) {    
    // Base case: if we have reached the end of our number, print the word
    if numeral >= 7 {
        print("\(word)")
        return
    }
    
    let index = phoneNumber.index(phoneNumber.startIndex, offsetBy: numeral)
    let digit = String(phoneNumber[index])
    let letters = numbersToLetters[digit]!
    
    if !letters.isEmpty {
        for letter in letters {
            word.remove(at: index); word.insert(letter, at: index)
            calculateWords(phoneNumber, numeral: numeral + 1)
        }
    }
}

calculateWords(number, numeral: 0)
```

## Boggle
### **Problem**:  Solve Boggle (find all words in a grid of letters)

*Discussion*: This problem can be thought of similarly to the paint-fill algorithm. We iterate over each tile in the grid to act as the starting letter, and then traverse each surrounding child. For each child, we must then traverse all sub-children, and so on. One of the key things to observe here is that we must have a visited (or 'closed') list of some kind, so that we can avoid revisiting tiles we've already used.

*Notes*: The code below omits the dictionary lookup function (`isValidWord()`), which can be implemented in whatever way is appropriate for the target platform.

*Recursive solution*
```swift
let grid = [["A", "B", "X"],
            ["E", "L", "T"],
            ["D", "Z", "U"]]
let gridSize = 3

struct Pos: Equatable {
    let x,y: Int
    static func == (lhs: Pos, rhs: Pos) -> Bool { return lhs.x == rhs.x && lhs.y == rhs.y }
}

// We check all possible paths, considering each tile as a valid starting point
for x in 0..<gridSize {
    for y in 0..<gridSize {
        let startingLetter = grid[x][y]
        
		// Here is the function which recurses through the graph, we 
		// pass a position, the current word, and a list of visited tiles.
		// The function iterates over all of the neighboring tiles, and
		// if the tile hasn't already been visited, we check if it's a valid
		// word and continue exploring the path from that tile
		
        func recurse(_ x: Int, _ y: Int, word: String, visited: [Pos]) {
            for xi in -1...1 {
                for yi in -1...1 {
                    guard xi != 0 || yi != 0 else { continue }
                    let nx = x + xi
                    let ny = y + yi
                    guard nx >= 0 && ny >= 0 && nx < gridSize && ny < gridSize else { continue }
                    guard !visited.contains(Pos(x: nx, y: ny)) else { continue }
                    
                    let newWord = word + grid[nx][ny]
                    if isValidWord(newWord) {
                        print("\(newWord)")
                    }
                    
                    recurse(nx, ny,
                            word: newWord,
                            visited: visited + CollectionOfOne(Pos(x: nx, y: ny)))
                }
            }
        }
        
        recurse(x, y, word: startingLetter, visited: [Pos(x: x, y: y)])
    }
}
```

*Output*: 
```
ABLE
ALE
BALE
BELT
BED
BLED
LAB
LED
DEAL
DEALT
ZEAL
```

---

## Author

**Matt Reagan** - Website: [http://sound-of-silence.com/](http://sound-of-silence.com/) - Twitter: [@hmblebee](https://twitter.com/hmblebee)


## License

The code examples here are for instructive purposes only. All content herein is released however under the GNU General Public License, Version 2.
