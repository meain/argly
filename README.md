argly
===========
A flexible and simple command line arguments parser that generates friendly help messages.

# Installation
```bash
npm install argly --save
```

# Usage

```javascript
// Create a parser:
var parser = require('argly').createParser(options);
var parsed = parser.parse(argsArray);

// parsed will be an object with properties corresponding to provided arguments
```

## Simple Example

Parse arguments provided by `process.argv`:

Given the following JavaScript code to parse the args:
```javascript
// Create a parser and parse process.argv
require('argly').createParser({
        '--foo -f': 'boolean',
        '--bar -b': 'string'
    })
    .parse();
```

And the following command:
```bash
node app.js --foo -b b
```

The output will be:

```javascript
//Output:
{
    foo: true,
    bar: 'baz'
}
```

You can also parse your own array of arguments instead of using `process.argv`:

```javascript
// Create a parser and parse provided args
require('argly').createParser({
        '--foo -f': 'boolean',
        '--bar -b': 'string'
    })
    .parse(['--foo', '-b', 'baz']);

//Output:
{
    foo: true,
    bar: 'baz'
}
```

You can also be more descriptive and add usage, examples, error handlers and validation checks:
```javascript
// Create a parser:
require('argly')
    .createParser({
        '--help': {
            type: 'string',
            description: 'Show this help message'
        },
        '--foo -f': {
            type: 'string',
            description: 'Some helpful description for "foo"'
        },
        '--bar -b': {
            type: 'string',
            description: 'Some helpful description for "bar"'
        }
    })
    .usage('Usage: $0 [options]')
    .example(
        'First example',
        '$0 --foo hello')
    .example(
        'Second example',
        '$0 --foo hello --bar world')
    .validate(function(result) {
        if (result.help) {
            this.printUsage();
            process.exit(0);
        }

        if (!result.foo || !result.bar) {
            this.printUsage();
            console.log('--foo or --bar is required');
            process.exit(1);
        }
    })
    .onError(function(err) {
        this.printUsage();
        console.error(err);
        process.exit(1);
    })
    .parse();
```

Running the above program with the `--help` argument will produce the following output:

```
Usage: args [options]

Examples:

  First example:
     args --foo hello

  Second example:
     args --foo hello --bar world

Options:

  --help Show this help message [string]

--foo -f Some helpful description for "foo" [string]

--bar -b Some helpful description for "bar" [string]
```


## Aliases

Aliases can be provided as space-separated values for an option:
```javascript
// Create a parser:
var parser = require('argly').createParser({
    '--foobar --foo -f': 'string', // "--foobar" has two aliases: "--foo" and "-f"
    '--hello -h': 'string',        // "--hello" has one alias: "-h"
});

parser.parse('--foo FOO -h HELLO'.split(' '));
// Output:
{
    foobar: 'FOO',
    hello: 'HELLO'
}

// **NOTE**: Only the first entry is used to determine the target property name--not the aliases.
```

## Booleans

An argument value of "true" or "false" is automatically converted to the corresponding boolean type. If a argument is prefixed with "no-" then it will be set to `false`.

```javascript
// Create a parser:
var parser = require('argly').createParser({
    '--foo': 'boolean',
    '--bar': 'boolean'
});

parser.parse('--foo --no-bar'.split(' '));
// Output:
{
    foo: true,
    bar: false
}
```

## Arrays

Any argument with multiple values will result in an `Array` value, but if you want to force an array for a single value then you can append "[]" to the option type as shown in the following sample code:
```javascript
// Create a parser:
var parser = require('argly').createParser({
    '--foo': 'string[]'
});

parser.parse('--foo a'.split(' '));
// Output:
{
    foo: ['a']
}

parser.parse('--foo a b c'.split(' '));
// Output:
{
    foo: ['a', 'b', 'c']
}
```

## Wildcards

A parser will throw an error for unrecognized arguments unless wildcards are used as shown in the examples below.

```javascript
// Create a parser:
var parser = require('argly').createParser({
    '--foo -f *': 'string[]' // Any unrecognized argument at the beginning is an alias for "foo"
});

parser.parse('a b --foo c'.split(' '));
// Output:
{
    foo: ['a', 'b', 'c']
}
```

```javascript
// Create a parser:
var parser = require('argly').createParser({
    '*': null
});

parser.parse('a b --foo FOO --bar BAR'.split(' '));
// Output:
{
    '*': ['a', 'b'],
    foo: 'FOO',
    bar: 'BAR'
}
```

## Complex Types

Square brackets can be used to begin and end complex types:

```javascript
// Create a parser:
var parser = require('argly').createParser({
    '--foo -f': 'boolean',
    '--plugins --plugin -p': {
        options: {
            '--module -m *': 'string',
            '-*': null
        }
    }
});

var parsed = parser.parse('--foo --plugins [ --module plugin1 -x -y ] [ plugin2 -z Hello ]'.split(' '));

// Output:
{
    foo: true,
    plugins: [
        {
            module: 'plugin1',
            x: true,
            y: true
        },
        {
            module: 'plugin2',
            z: 'Hello'
        }
    ]
}
```


# Similar Projects

* [optimist](https://github.com/substack/node-optimist) - Popular but deprecated. Awkward API and not DRY as shown in the following comparison:

__optimist:__

```javascript
var result = require('optimist')(args)
    .alias('h', 'help')
    .describe('h', 'Show this help message')
    .boolean('h')
    .alias('f', 'foo')
    .describe('f', 'Some helpful description for "foo"')
    .string('f')
    .alias('b', 'bar')
    .describe('b', 'Some helpful description for "bar"')
    .string('b')
    .argv;
```

__argly:__

```javascript
var result = require('argly')
    .createParser({
        '--help':   { type: 'string', description: 'Show this help message' },
        '--foo -f': { type: 'string', description: 'Some helpful description for "foo"' },
        '--bar -b': { type: 'string', description: 'Some helpful description for "bar"' }
    })
    .parse();
```

* [yargs](https://github.com/chevex/yargs) - A fork of `optimist` with documentation for those who speak Pirate.
* [minimist](https://github.com/substack/minimist) - Very few features (by design). Not DRY.

# TODO

* Support equal separator: `--hello=world`
* Support number arg: `-x256`

# Additional Reading

For module help, check out the test cases under the "test" directory.

# License

MIT
