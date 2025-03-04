DEPRECATED
===

Node.js v11.13+ includes most of this functionality in its [v8.writeHeapSnapshot API](https://nodejs.org/api/v8.html#v8writeheapsnapshotfilenameoptions).

The only thing missing is being able to trigger it from a signal by default, for that, please use https://github.com/Jimbly/node-heapdump-signal

node-heapdump-prebuild
===

Fork of `node-heapdump` with npm prebuild support to avoid deployment-time
g++/make dependencies.

Currently should provide prebuilds for Node v12.19.1, v14.8.0, v16.13.0, v18.20.4 (Maintenance LTS), v20.18.0 (Active LTS), and v22.11.0 (Current)
Ping me if you need other versions, or send a PR that publishes all of them automatically or something :).


node-heapdump
==

Make a dump of the V8 heap for later inspection.

### Install

    npm install heapdump-prebuild

### Usage

Load the add-on in your application:

    var heapdump = require('heapdump-prebuild');

The module exports a single `writeSnapshot([filename], [callback])` function
that writes out a snapshot.  `filename` defaults to
`heapdump-<sec>.<usec>.heapsnapshot` when omitted.

    heapdump.writeSnapshot('/var/local/' + Date.now() + '.heapsnapshot');

The function also takes an optional callback function which is called upon
completion of the heap dump.

    heapdump.writeSnapshot(function(err, filename) {
      console.log('dump written to', filename);
    });

The snapshot is written synchronously to disk.  When the JS heap is large,
it may introduce a noticeable "hitch".

On UNIX platforms, you can force a snapshot by sending the node.js process
a SIGUSR2 signal:

    $ kill -USR2 <pid>

The SIGUSR2 signal handler is enabled by default but you can disable it
by setting `NODE_HEAPDUMP_OPTIONS=nosignal` in the environment:

    $ env NODE_HEAPDUMP_OPTIONS=nosignal node script.js

Catch the SIGUSR2 signal to write the snapshot to a custom location:

```js
if (!/nosignal/.test(process.env.NODE_HEAPDUMP_OPTIONS)) {
  process.on("SIGUSR2", function() {
    heapdump.writeSnapshot('/var/local/' + Date.now() + '.heapsnapshot');
  });
}
```

### Inspecting the snapshot

Open [Google Chrome](https://www.google.com/intl/en/chrome/browser/) and
press F12 to open the developer toolbar.

Go to the `Memory` tab, right-click in the tab pane and select
`Load profile...`.

Select the dump file and click `Open`.  You can now inspect the heap snapshot
at your leisure. Some snapshots may take a long time to load, on the order of
minutes or even hours.

Note that Chrome will refuse to load the file unless it has the `.heapsnapshot`
extension.

### Caveats

On UNIX systems, the rule of thumb for creating a heap snapshot is that it
requires memory twice the size of the heap at the time of the snapshot.
If you end up with empty or truncated snapshot files, check the output of
`dmesg`; you may have had a run-in with the system's OOM killer or a resource
limit enforcing policy, like `ulimit -u` (max user processes) or `ulimit -v`
(max virtual memory size).
