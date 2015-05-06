# Test plan for latest changes

## TODO

* Recently checked all of the text files from all of the trivial cores.
  Manually played around with the agent core and it looks fine.  Checked
  everything EXCEPT "nongarbage" from the agent and supervisor coes.
* Need check "nongarbage" from both agent and jobsupervisor cores.
* Consider adding support for JSRegExp?
* Document the changes themselves (as would appear in the PR text)
* Re-run the whole test plan, including cstyle, automated checks, etc?
* Make sure to run the automated tests using core files from older Nodes?
  (I think there's a way to do that?)

## Three basic test cases

* "explicit-abort": a script that just calls `process.abort()` immediately at
  the top level
* "defer-abort": a script that calls `process.abort()` in a setImmediate
  callback.
* "gcore-loop": a script that calls several functions, the last of which goes
  into an infinite loop with a known object on the stack


# Results

## Sanity checks

* All expected 32-bit core files are 32-bit
* All expected 64-bit core files are 64-bit
* All v0.10.30 core files are V8 version 3.14.5.9
* All v0.12.2 core files are V8 version 3.28.73.0

## Testing the change

    mdb -e "::load ../mdb_v8.so; \
        ::findjsobjects -a -k badprop | ::findjsobjects | \
        ::jsprint -b -v"

Core files: (32-bit, 64-bit) x (v0.10.30, v0.12.2) x (three test cases above)

The above command shows the objects that have been filtered by this change.  For
each of the test cases, I checked the output of the above for any objects that
appeared valid, and didn't find any.


## Manual regression tests

    mdb -e "::load ../mdb_v8.so; \
        ::findjsobjects -l | ::findjsobjects | \
        ::jsprint -b -v -N 256"

Core files: (32-bit, 64-bit) x (v0.10.30, v0.12.2) x (three test cases above)

The above command prints the contents of all objects visible to findjsobjects,
using the default jsprint depth of 2 and a maximum jsprint buffer size of 256
(which is used to avoid printing giant strings representing file contents, which
mess up heuristics for guessing).  On the output files, there were no matches
for the text "warning" or "error".  I looked for warnings from the debugger
(identified by "&lt;" characters that aren't part of "<anonymous>" or "<anon>"),
and found only:

* problems with array elements (which aren't fixed by this change, but also
  aren't searchable with findjsobjects)
* problems related to specific errors that we're choosing to ignore (e.g.,
  warnings about accessors)

Given that this change attempts to remove garbage, it's not the end of the world
if we don't properly remove _all_ garbage, as long as we're making things
better.


## Automated regression tests

The automated regression tests ran fine on both the v0.10.30 and v0.12.2.


## cstyle

cstyle.pl with no options passes on all changed files.
