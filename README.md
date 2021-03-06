
# Crane Parser
### Generate fast, push-based LR parsers for JavaScript!

## Install

### From NPM

```sh
$ npm install -g crane-parser
```

### From source

```sh
$ git clone https://github.com/gabrielratener/crane-parser
$ cd crane-parser
$ npm install -g
```

## Usage

A Crane grammar often specifies multiple top-level non-terminals. With Crane we get to chose which one we want to use to generate our parser.

### Basic Usage
```sh
# by default use non-terminal of first production in file as root
$ crane /path/to/grammar.crane > /path/to/parser.js
```

### Specify Root
```sh
# specify `expression` as root of grammar
$ crane -r expression /path/to/grammar.crane > /path/to/parser.js
```

## Grammar Language

One of the defining features of Crane is its high-level grammar definition language. In addition to being able to specify top-level productions, Crane introduces scoped productions with a namespacing system to reference non-terminals across scopes. Crane also has a hygienic macro system to simplify grammar syntax and prevent superfluous productions.
The Crane Language is indentation-based and uses LiveScript functions to define actions.

### Basic Arithmetic Expression Grammar Example
```ls
# set precedence (lowest at top, highest at bottom)
# %<associativity> '<token type 1>' '<token type 2>' ...
%left '+' '-'
%left '*' '/'
%none unary     # for unary + or -
%right '^'

expression
    > \number                   => parseFloat $[0]
    > '(' expression ')'        => $[1]

    > expression '^' expression => $[0] ** $[2]

    > expression '*' expression => $[0] * $[2]
    > expression '/' expression => $[0] / $[2]

    > expression '+' expression => $[0] + $[2]

    # livescript code snippets can also be multiline!
    > expression '-' expression =>
        a = $[0]
        b = $[2]
        a + b
    
    %prec unary
    > '+' expression            => +$[1]

    %prec unary
    > '-' expression            => -$[2]

root
    # Without a code snippet the production evaluates to the first symbol
    > expression

```

### Basic Rules

As you see in our above grammar we are declaring two non-terminals, _expression_ and _root_. We then define a scope in which we specify productions for _expression_. These productions begin with a `>`, and can optionally end with a livescript code snippet to run upon our production being matched. If the livescript snippet (e.g. `=> ...`) is omitted, the resolution value of the match becomes either the value of the first symbol in the match, or if it is an empty production (e.g. `> ''`), it resolves to an empty array.

The symbols in our rules must be explicitly declared as non-terminals or terminals. A terminal symbol mostly follows livescript string syntax, so it can be declared with a forward slash (e.g. `\token-type`), or surrounded by quotes. Non-terminal symbols are simply identified by their name much like a JS variable.

A symbol (terminal, or non-terminal) follows JS variable naming rules, except it cannot contain `$`, and it may contain dashes (`-`).

## Parser Usage

### Basic Usage Example

```js
import {Parser} from "path/to/my/generated/parser";

// initialize our parser with our context
const parsing = new Parser({
    // these are the options we pass into the parser instance ...


    // `context` is an object that will take the value of `this` inside our livescript snippets
    context: {
        // We can define variables and functions here that we want to access inside our parser!
    }
});

// note that Crane has no lexing capabilities
// it is up to you to tokenize your code :(
for (const token of lex(codeToBeLexed)) {

    // This parser is push-based!!!
    // parse at your own pace
    parsing.push(token);
}

// when done pushing we call `finish` to reap the rewards of our parsing
const parsingResult = parsing.finish();
```

### Options

* #### `context`

    This is the `this` value that the livescript actions receive during parsing

* #### `context`


### API

As you can see above, the parser has two functions that we use for the actual parsing `push`, and `finish`.

#### `push(token)`

Use this to push proper tokens (see below), to the parser instance.
If an invalid or unexpected token is pushed into the parser, this will throw an error (see errors).

#### `finish()`

Use this when done parsing.
Calling `finish` will return the result of your parsing when called appropriately. If called prematurely it will throw a premature end error. This means the existing sequence of symbols in the parsing instance doesn't represent a valid string in your language.

### Token Format

Since Crane doesn't have a built-in lexer (for now), you must make sure the tokens being fed to the parser are in the proper format.

To Crane a token looks like this

```js
{
    // the terminal type (e.g. `> "what-you-would-see-here" "other-terminal" some-non-terminal`)
    // must follow the naming rules specified above
    type: <string>

    // The value the token represents
    // By default this value is what gets fed into the livescript snippets in place of the symbol
    value: <whatever>

    // Location (loc) is optional, but if specified must look like this:
    loc?: {
        start: {
            // line offset of token start, starting at 1
            line: <int>

            // column offset of token start, starting at 0
            column: <int>
        }

        // same deal as start but for end of token!
        end: {
            line: <int>
            column: <int>
        }
    }
}
```

For example a `number` token that we would feed to a parser generated by the grammar above could looks like this

```js
{
    type: 'number',
    value: '122.5',
    loc: {
        start: {
            line: 1,
            column: 5
        },
        end: {
            line: 1,
            column: 10
        }
    }
}
```

## Errors

### Check back soon!

# Yes, I know this is incomplete...
## Help and suggestions are always welcome
