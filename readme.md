 Crash Analyzer Support

This package enables your application to produce stack traces in a host that supports the
[V8 stack trace API](https://v8.dev/docs/stack-trace-api) which can be decoded using the Microsoft Edge Crash Analyzer
experimental tool. This tool uses the Symbol Server integration in Microsoft Edge DevTools, plus Enhanced
Unminification, to provide a way both to view an unminified call stack, and to see the original source. This
functionality is based on using TypeScript as the source code language.

## Example

This is an error we have tracked from Microsoft Edge:

```
RangeError: Maximum call stack size exceeded
    at new q (devtools://devtools/bundled/entrypoints/heap_snapshot_worker/heap_snapshot_worker.js:1:38478)
    at K.createEdge(devtools://devtools/bundled/entrypoints/heap_snapshot_worker/heap_snapshot_worker.js:1:32931)
    at G.edge (devtools://devtools/bundled/entrypoints/heap_snapshot_worker/heap_snapshot_worker.js:1:6912)
    at G.name (devtools://devtools/bundled/entrypoints/heap_snapshot_worker/heap_snapshot_worker.js:1:6267)
    at f (devtools://devtools/bundled/entrypoints/heap_snapshot_worker/heap_snapshot_worker.js:1:30931)
    at devtools://devtools/bundled/entrypoints/heap_snapshot_worker/heap_snapshot_worker.js:1:31513
    at devtools://devtools/bundled/core/platform/platform.js:1:330
    at r (devtools://devtools/bundled/core/platform/platform.js:1:376)
    at r (devtools://devtools/bundled/core/platform/platform.js:1:425)
    at r (devtools://devtools/bundled/core/platform/platform.js:1:425)

Source modules:
    devtools://devtools/bundled/entrypoints/heap_snapshot_worker/heap_snapshot_worker.js 9e8af998e1e1bbdb3ede85775d2c64825ccaff4b835c8b4238f10952b3890144
    devtools://devtools/bundled/core/platform/platform.js 9a35dc3d31ba75643a6248e885e91caf800e4a293284695d1e96ab519dc563b2
```

After connecting to Symbol Server, this can be unminified to:

```
RangeError: Maximum call stack size exceeded
    at constructor for JSHeapSnapshotEdge (devtools://devtools/bundled/entrypoints/heap_snapshot_worker/HeapSnapshot_edge.ts:2767:5)
    at K.createEdge(devtools://devtools/bundled/entrypoints/heap_snapshot_worker/heap_snapshot_worker.js:1:32931)
    at HeapSnapshotRetainerEdge.edge (devtools://devtools/bundled/entrypoints/heap_snapshot_worker/HeapSnapshot_edge.ts:256:42)
    at HeapSnapshotRetainerEdge.name (devtools://devtools/bundled/entrypoints/heap_snapshot_worker/HeapSnapshot_edge.ts:208:17)
    at compareEdgeFieldName (devtools://devtools/bundled/entrypoints/heap_snapshot_worker/HeapSnapshot_edge.ts:2156:17)
    at compareNodeAndEdge (devtools://devtools/bundled/entrypoints/heap_snapshot_worker/HeapSnapshot_edge.ts:2199:18)
    at partition (devtools://devtools/bundled/core/platform/array-utilities.ts:37:9)
    at quickSortRange (devtools://devtools/bundled/core/platform/array-utilities.ts:53:25)
    at quickSortRange (devtools://devtools/bundled/core/platform/array-utilities.ts:55:5)
    at quickSortRange (devtools://devtools/bundled/core/platform/array-utilities.ts:55:5)
```

You can see that this is already a substantial improvement. But if your source code is additionally included in your
source maps, Edge DevTools can also display your original source in the Sources tool and highlight the line indicated
by the mapped location.

# Requirements

This library requires a Chromium-based browser at version 101 or newer, or Node.js version 19 or newer. It can be used
in other environments, but will have no effect.

# Usage

How you obtain one of these stack traces is up to you. Many telemetry libraries such as Azure App Insights can capture
unhandled error information from your JavaScript applications. In the [web](./demo/web) and [Node.js](./demo/node)
examples, we illustrate listening to `window.onerror` or `process.on('uncaughtException')` events, or their
corresponding `Promise`-based events, respectively.

Once you have a call stack with the `Source modules` section, copy it into your Edge DevTools Crash Analyzer tool, and
press Ctrl+Enter or click the Analyze button. More information about using the Crash Analyzer tool can be found in the
[Microsoft Edge documentation](https://go.microsoft.com/fwlink/?linkid=2228026&clcid=0x409).

One note: Crash Analyzer requires that your source maps must be accessible from DevTools. That means that they either
must have already been cached (loaded, for example, by a `sourceMappingURL` comment) or via Symbol Server. You can
learn more about making your source maps securely available via Symbol Server [here](https://learn.microsoft.com/en-us/microsoft-edge/devtools-guide-chromium/javascript/publish-source-maps-to-azure).
