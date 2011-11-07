## Fiat: the auto-Make-er

Fiat's a bit like [autotest](http://www.zenspider.com/ZSS/Products/ZenTest/) for `make`-managed projects.

Fiat monitors a `make` task's dependencies. Each time one of those dependencies changes, fiat executes the task, prints the results, and reports whether the process succeeded or failed. It can be used to check program output or run tests without having to manually execute the task.

## Installation

Fiat depends on ruby and rubygems, so make sure they're installed. Then:

	$ gem install fiat

If you haven't installed any gems before, you *may* need to add rubygems' installation directory to your `$PATH`.

## A Usage Example

Suppose we're developing a little C program, and we'd like to automatically run it each time we make changes.

Sample Makefile:

	all: hello_world.c
		gcc -o hello hello_world.c
	run: my_proj
		./hello
		
We can execute:

	$ fiat run
	Running "make run" on changes...
	gcc -o hello hello_world.c
	./hello
	Hello, world!
	######################################## // <- this line should be green =)
	...

Each time we save changes to `my_proj.c`, fiat will execute `make run` and print the results, followed by a line of hashes. This line is <span style="color: green;">green</span> if the program exited cleanly (with exit status == 0) and didn't include one of the failure terms (see below) or <span style="color: red;">red</span> if the program failed in some way. This makes it easy to tell at a glance whether our tests succeeded or failed.

If the output of a task includes one of the defined key terms (`failure_terms`), fiat interprets that run to have been a failure and prints the end-of-run bar in red.

## .fiatrc

In the same directory as the Makefile, we can optionally define a `.fiatrc` file which is interpreted when fiat is executed. In it we can set two variables:

- `instruction` -- the `make` task to be run. Defaults to `"test"`.
- `failure_terms` -- a list of strings that imply failure if any of them appear in the program's output. Defaults to `["failed", "error"]`.

For example, the `.fiatrc` for the sample project above might look like:

	failure_terms = []
	instruction = "run"

Since we're not using a testing package, there's (probably) no reason to define `failure_terms`; we'll only get a red end-of-run bar when the program returns a non-zero exit status.  However, the `make` task we're using is `run`, so setting `instruction = "run"` allows us to run fiat without arguments:

	$ fiat
	Running "make run" on changes...
	gcc -o hello hello_world.c
	./hello
	Hello, world!
	########################################
	...

Alternately, suppose we're using `make` to manage an Erlang project with EUnit tests, and we'd like to know when our tests fail. The `make` task to execute our tests is fiat's default, `test`.  For fiat's purposes, we'd like to define a failed test to be one in which the term "Failed:" appears.

So, our `.fiatrc` is just:

	failure_terms = ["Failed:"]

Easy-peasy.