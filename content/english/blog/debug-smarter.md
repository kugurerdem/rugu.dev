---
title: 'Debug Smarter'
draft: false
date: '2024-08-25'
---

In this essay, I share some tips that I've found particularly beneficial for my
own debugging experience. Hope it helps the reader as well.

If you have any cool tips you'd also like to share, feel free to share them. I
might include them in the essay and give you credit for it.

# Writing Logs to STDERR

I've noticed that developers, myself included, often instinctively write debug
logs to STDOUT. While this might not cause problems if the program's output
isn’t intended for other programs to consume, it becomes crucial when managing
multiple scripts that depend on each other (such as when one script writes to a
file that another reads).

In these situations, if you're writing everything to STDOUT, you might not be
taking full advantage of the pipe and redirection mechanisms available in your
UNIX environment. The primary purpose of the STDOUT file descriptor is to
facilitate interprocess communication through mechanisms like piping. By using
STDOUT for debug logs and similar outputs, you might be accidentally
complicating your own workflow and missing out on its intended purpose.

So, what to do instead? Simply write your debug logs to STDERR instead of
STDOUT. Since both STDOUT and STDERR are displayed in your terminal, you'll
still see your debug logs on the terminal screen as usual.

Many common UNIX utilities already follow this practice. Run grep on a
non-existent file, you will see that the error message is sent to STDERR, while
any matching lines would be sent to STDOUT. Attempt to create a directory with
`mkdir` that already exists or run `cat` on a non-existent file, the error
messages are directed to STDERR. The same behavior can be observed with `curl`
and `find` when they encounter errors. These programs consistently write error
and debug messages to STDERR.

Simply put, if the output of your process isn’t intended for use by another
process, direct it to STDERR instead. Additionally, if you want certain errors
to be manageable by other programs, you can use EXIT CODES. This allows other
programs to decide how to handle specific error situations.

Just don't clutter the STDOUT.

# Conditional Debugging

In NodeJS, there is this function called
[debuglog](https://nodejs.org/docs/latest/api/util.html#utildebuglogsection-callback)
under the default util module.

What it does is to return you a debug function that conditionally logs to
STDERR, based on the value of `NODE_DEBUG` env variable.

To give you an example, consider the following file `debug.js`:

```javascript
const
    {debuglog} = require('util'),

    normalLog = debuglog('normal'),
    verboseLog = debuglog('verbose'),

    main = () => {
        normalLog('This is a normal log!')
        verboseLog('This is a more specific, detailed log!')
    }


main()
```

Now, on your shell, try running `node debug.js`, you won't get any results
printed to terminal.

Now, try it like `NODE_DEBUG=normal node debug.js`;

```terminal
$ NODE_DEBUG=normal node debug.js
NORMAL 16231: This is a normal log!
```

Now, try it like `NODE_DEBUG=normal,verbose node debug.js`;

```terminal
$ `NODE_DEBUG=normal,verbose node debug.js`;
NORMAL 16337: This is a normal log!
VERBOSE 16337: This is a more specific, detailed log!
```

I hope you see the value of this approach. By adjusting the value of the
`NODE_DEBUG` environment variable, you can control the level of detail you want
to see in your debug logs. You can also have different debug logs for unrelated
parts of the code so that you can seperate unrelated logs from each other. What
I generally like to do is to have different debug logs for different modules.

This way, you can have more control over your debug logs so that they don't
clutter the STDERR much. You can, for example, choose to disable/enable certain
kind of debug logs depending on what you really want to see at that moment.

The good thing is that the way `debuglog` works under the hood is pretty
straightforward, (see [source
code](https://github.com/nodejs/node/blob/43f699d4d2799cfc17cbcad5770e1889075d5dbe/lib/internal/util/debuglog.js#L87)).

If it already doesn'n exist, you can easily implement a similar mechanism in
your favorite language (as long as it supports higher-order functions). Feel
free to translate the following custom JavaScript code I wrote (for
demonstration purposes) into the language of your choice.

```javascript
const debuglog = (logName) => {
    const logNames = process.env.NODE_DEBUG?.split(',') || []

    if (logNames.includes(logName))
        return log => process.stderr.write(`[${logName}] ${log}\n`)

    return () => {} // noop
}
```

# Tap Functions

A tap function is a function that returns the value it receives while executing
specific side effects. It is commonly used for logging, particularly for
intermediate values, without requiring significant code refactoring.

Let me show you how it looks. Consider the following piece of code I came up
with:

```javascript
f1(f2(f3( ...someArgs)))
```

Say you want to see what the function `f3` returns in your debug logs. What
would you do?

If you were to write something like:

```javascript
const f3Result = f3(...someArgs)
log(f3Result)
f1(f2(f3Result)
```

You can probably understand the frustration that comes with this approach.
However, tap functions let you do something like this instead:

```javascript
f1(f2(taplog(f3( ...someArgs))))
```

As you can see, we simply wrapped the `f3` function call with `taplog`. This
approach requires much less effort and is especially helpful when you need to
quickly log an argument passed to another function (in this case, the result of
`f3` being passed to `f2`) without altering the code structure much.

I find tap functions particularly helpful when working with frameworks like
`React`, where you occasionally pass expressions as arguments to other components
and want to log those expressions without disrupting the code flow.

What I like is to create a generic function for creating tap functions, called
`tap`.

```javascript
const tap = fn => (value) => {
    fn(value)
    return value
}
```

And then use it to create your specific tap functions based on your needs.

```javascript
const
    {debuglog} = require('util'),

    debug = debuglog('blabla'),

    tapDebug = tap(debug),
    tapLog = tap(console.log),
    tapErr = tap(console.error),

    tapAll = tap((val) => {
        tapDebug(val)
        tapLog(val)
        tapErr(val)
    })
```

These are just example functions, but the main idea here is that you can
basically create your own tap functions for your own needs.
