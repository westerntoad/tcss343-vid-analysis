# Chess Programming: Board Representation

## Introduction

> *video of me talking over the script*

As a Computer Science student, there are many aspects of software engineering that I find beautiful. The struggle to understand a difficult problem, to analyze it, and finally arrive at a correct solution is enormously satisfying. Additionally, the development and application of new techniques, such as Machine Learning, allow problems to be solved that were once deemed unsolvable. Software development is a world of enormous complexity, with many avenues and side streets to get lost in.

To me personally, I find the minimalistic aspects of programming to be the most elegant. When scrutinizing over the smallest aspects of a piece of code, it really puts into perspective how, as a whole, these grandiose codebases owned by billion-dollar companies really do sum up to just arrays of binary switches on a piece of hardware represented as ones and zeros. Today, I'd like to showcase this austere perspective on coding via one of my favorite representations: chess.

## Why Chess?

> *plain abstract animated background*

If the sole intention of this video was to present the software representation of a board game with a straightforward rule set, what makes chess special as opposed to games like Connect Four, Tic Tac Toe, or Checkers?

> *figures of game pieces show up on the background*

The principal reason why I find chess more appropriate than the aforementioned examples is because all of them are deterministic in nature; that is, if the algorithm has been implemented properly, one can accurately ascertain whether they will win with one hundred percent certainty at any given point during play. The most notable of the previous examples is Tic Tac Toe, which has an algorithm simple enough to be performed by a human; and, if done properly, will always allow you to win if you play first.

> *an animation of a sample game of tic-tac-toe*

There are also additional reasons that make chess even more provocative compared to other unsolved strategy games like Shogi or Go, and that is the board.

> *a grid of 8x8 chess squares appear*

The fact that a chess board is made up of an eight-by-eight grid of cells has notable implications for its implementation in code. Not only is each dimension of the grid a power of two, but the total number of squares has a product of 64, which is small enough to be operated on within a single instruction on processors with a 64-bit architecture. The importance of this detail becomes more apparent when considering specific representations of the board on a machine, which will be discussed in more detail later in this video.

> *annotations to the grid are added, counting each dimension and their total number of squares.*

## Board Representation: Mailbox & Piece Lists

> *video of me talking through the script*

Before any fancy move generation or evaluation functions, we first need to get our pieces in play with a board representation. Primarily, this is an issue of selecting the appropriate data structure. There are many different approaches, and I would like to start with - in my opinion - the most intuitive: the Mailbox approach.

> *diagram of the 8x8 matrix*

In the mailbox approach, each cell on the board has it's own addressable space in memory, usually represented as a two-dimensional matrix. This technique is square-centric; where, pieces are primarily addressed and stored in this matrix, and indexing is done in the same way you would visually look at a chess board: ~ take a file and rank, and return the piece's data ~ . This approach is the recommended strategy for newcomers to chess programming.

> *pieces popping up in each square individually within the matrix*

While the mailbox approach uses a square-centric technique, we can use the inverse association of a piece-centric approach with Piece-Lists. In this strategy, we take a basic list or array data structure in code, and store all of the pieces still left on the board, each associated with a specific rank and file. This makes iterating through pieces to generate their legal movement much easier, as you don't need to spend extra computational power checking possibly empty cells within a matrix, such as with the mailbox approach. However, on the opposite end, indexing the destination squares of those pieces in a piece-centric approach requires iterating over enemy pieces, which would be a constant lookup for a square-centric representation.

> *a list-like diagram with pieces showing up in the list, and then on the chess board with indices of each piece*

> *then, a video of me talking through the script*

Both of these approaches have other surpluses and deficits that I have not mentioned, as well as hybrid approaches that make use of the upsides of both data structures. However, there is one strategy that is by far the most popular way to represent a set of pieces on a chess board, and that is with Bitboards.

## Board Representation: Bitboards

> *video of me talking through the script*

Bitboards are piece-centric, similar to the piece-list technique; however, instead of using composite data structures for each individual piece, we take advantage of the computer's hardware, set theory, and bit manipulation tactics to store pieces of a specific classification within single 64-bit variables.

> *showing a set and it's associated chess board*

A board has exactly 64 squares. We start our process by classifying a single finite set of exactly 64 entries, with each entry being associated with a square on the board and a binary state of either 1 or 0. Each entry's value represents a square's *vacancy*. So, for example, if we say that the set as a whole represents the squares on a board that a knight occupies, the start of the game might look like this:

> *the set and associated chess board appears the same as the starting squares of the knight*

By performing set albebra on sets of differing classifications, we can represent more complex relations. For example, suppose we have two sets, A and B. A is the vacancy set of all black pieces on the board, and B is a vacancy set of all rooks on the board. By taking the set intersection of both A and B, we are, by definition returning everything in common of both sets. In this case, it means the intersection of sets A and B shows all black rooks on the board. In fact, this type of set intersection is a very common operation that will occur, considering the manner in which we define our position's bitboards.

> *showing what the set intersection would look like*

This set's most common hardware representation is a single, 64-bit unsigned integer. This integer is what we refer to as a bitboard. Through the use of bitwise operators, it is possible to operate on bitboards in a similar way as a vacancy set. Because computers already store all data in a binary state, as well as the fact that most processing architectures support operations on variables of 64 bits, a bitboard approach lends itself to being extremely powerful in both speed and implementation.

> *something*

By this technique, only eight bitboards are necessary to fully represent the vacancy of every single piece on a chess board. Six are required for each type of piece: pawn, knight, bishop, rook, queen, and king. Then two more for each color: the black and white sides. Through set operations, we can determine all the information we need for evaluation and makemove functions down the line.

## Move Generation

Now that we have a way to represent all pieces on the board, it is possible for us to generate their movement. There are many strategies on how to generate the movement of pieces on a board, and what I will be discussing will be the generation of 'pseudo-legal' moves. Pseudo-legal moves are moves in which you generate moves while not taking into account the exact legality of the move - ignoring things such as king safety via pins and checks. I would personally categorize a chess piece's move generation algorithm into one of three categories: look-up pieces, sliding pieces, and pawns. The first category I would like to talk about are look-up pieces.

Look-up pieces are pieces in which it is not necessary to receive information from other pieces on the board - friendly or otherwise - to properly compute its pseudo-legal destination squares. Knights and kings are the only pieces categorized in this fashion. There are really only two feasible ways to implement a move generation algorithm for look-up pieces. The first is by calculation, where you perform bit shifting operations relative to the piece's origin square. The second option is to pre-calculate all destination squares indexed by its origin. In this situation, there is no real reason to choose to calculate the destination squares as opposed to doing it pre-calculated, unless there is very specific space implications. Implementing this is the most straightforward of the three categories of move generation algorithms and will not be discussed in further detail.

Similar to look-up pieces, pawns have a predetermined number of places they can move relative to their origin square; however, their movement does depend on the vacancy of both enemy and friendly neighboring squares. While it is possible to pre-calculate a destination-square mask relative to its location, the optimization implications compared to calculating it during runtime are either non-existent or neglible in their difference; and, in my opinion, not worth consideration in this video.

I would now like to talk about sliding pieces. Sliding pieces are characterized by their ability to cover varying distances across a chess board depending on the presence of both enemy and friendly pieces. The three sliding pieces are bishops, rooks, and queens. Sliding pieces are arguably the most complex algorithms out of the three categories, as there has been an enormous amount of effort in optimizing this algorithm for bitboards. The first approach a developer might take to implement it would be, for each direction valid for it's requisite piece, iterate through that direction until you either reach an enemy piece, a friendly piece, or the edge of the board. This does end up being somewhat of a naive approach, however. Through the work of many people smarter than the author of this video, you can actually return a sliding piece's destination squares with a single look-up operation through the use of something called Magic Bitboards.

## Magic Bitboards

A single lookup for every combination of origin square, friendly square, and enemy square input? If we were to have a single lookup table for every possible combination of these inputs, we would end up with astronomically large tables taking up quintillions of Zettabytes of space! In comparison, the total of both the knight and king lookup tables required a total of 1 Kilobyte of space. Luckily, there are many strategies in our toolbet we can use to optimize the space we require for our lookup table.

> *Notes for the size of lookup table:* \
> *Number of possible origin squares = 64* \
> *Number of possible friendly bitboards = 2^64* \
> *Number of possible enemy bitboards = 2^64* \
> *Number of bits used in each array entry = 64* \
> *Total = 64 * 64 * 2^64 * 2^64*

When casting the direction of a sliding piece, it is not entirely necessary to take in the input of both friendly and enemy pieces. Both friendly and enemy pieces act as blockers to the piece we are moving. The only difference is that if it were an enemy piece, the player can capture that piece while moving to that square, while a friendly piece cannot be captured. With this logic, we only need the origin square and a bitboard of *blockers* which hold the union of all friendly pieces and enemy pieces. We can pretend in this case that all blockers are capturable, and subtract all friendly pieces from the resulting attack squares. By doing this, we have reduced our space requirement to a mere eight Zettabytes! Once again, however, there is room for optimization.

> *Notes for the size of lookup table:* \
> *Number of possible origin squares = 64* \
> *Number of possible blocker bitboards = 2^64* \
> *Number of bits used in each array entry = 64* \
> *Total = 64 * 64 * 2^64*

Our next observation might be that our lookup table is indexing this array by many unneeded vacancy bits in our blocker bitboard. If we are indexing our lookup table, we are checking squares not even in the path of where our sliding piece might move. What we can do is create a mask over those squares, and index our lookup table by whether our masked squares are occupied. This *drastically* decreases the size required by our table, and now only requires two Megabytes!

> *Notes for the size of lookup table:* \
> *Number of possible origin squares = 64* \
> *Number of possible blocker bitboards (with bit mask) = 2^12* \
> *Number of bits used in each array entry = 64* \
> *Total = 64 * 64 * 2^12*

While two Megabytes is a perfectly reasonable amount of space to require for a lookup table, we unfortunately now have a new problem. When returning our masked bits, the possible configurations in which the spaces are occupied is spread across the entire 64-bit range of our masked blocker bitboard. The solution to this problem is through the use of a hashing algorithm! In essence, a hashing algorithm takes a *key* (our masked blocker-bitboard) and returns the *value* associated with that key (our destination squares). What the algorithm itself does is turn our key into an index with a smaller size. In our case, this allows us to flatten out our input into a size manageable enough to store in a table.

What is of utmost importance to us when implementing our hashing algorithm is speed. In order to ensure the speed of our lookups in our table, we must be certain that, when flattening (also called hashing) our key into an index within the table, that index does not match any other indices within that table. When this happens, it is called a collision; and, while not the end of the world, does have additional space and time implications within our move generation algorithm. If we were to make a single hashing function in which we are certain there will never be any collisions with other values, it is called a *perfect* hashing function. In our application, this is actually a very feasible expectation.

Our hashing function can actually be summarized into a single line of pseudocode.

> *(blockers · magic) >> (64 - index_size)*

While short, this has a lot going on here, so let me explain. First, 'blockers' is a bitboard of all occupied friendly enemy pieces masked by our original piece's possible destination squares. By its nature, this will be sparse in terms of the number of bits that are set to one as opposed to zero, due to the mask. If we add some entropy to this value by multiplying it by a large number of arbitrary value, more bits within the blockers bitboard will most likely be set to one, which has the potential to give us more of an abundant set of zeros and ones. This number we are multiplying the blocker bitboard by has special properties and is almost always a unique number for each origin square of our piece. Multiplying our blockers by some random number doesn't always give us the magic number we need, but once a set of values are found, it can be continued to be used throughout the application's lifespan. Through trial and error or through researching online, these numbers can be calculated or found and are actually cheap enough to calculate upon starting of your chess application.

Once we populate our bitboard, we then collapse all of the bits to the right, until we only get the most-significant (left-most) `index_size` number of bits of the word. While it is possible to shorten the `index_size` to a fewer number of bits, most of the time it will be twelve, which is the smallest number of bits our directional bit-mask will contain. 

There is an additional benefit to using this hashing algorithm that I have yet to mention. If you look at our move generation algorithm currently, you may notice we have some duplicity in our result set when entering different occupancy masks. Depending on the way you implement your hashing function, you can allow collisions of input with your occupancy mask as long as the destination squares return to the same value, which allows your tables to be even smaller than the two Megabytes as stated earlier! It does require some more rigorous testing to find good enough magic numbers (or magics) to reduce the size of your `index_size`, but it is an unintended benefit to our algorithm.

## Conclusion

There are many different ways you can write and develop the game of chess in code. This video is meant to take a peek at some necessary background if you wish to write a version of the game at a computationally efficient level; and, even then, there are many, many optimization improvements you can make for your internal representation to be even better than the explanations I gave in this video. I simply hope that this serves as an adequate introduction to that side of chess computing.

I hope you enjoyed the video, and have a wonderful day!
