# file_server_plus
This is a `deno` static file webserver, copied from: [file_server.ts](https://github.com/denoland/deno_std/blob/main/http/file_server.ts), with an additional optional command line arg: `--handler FILE.ts` to have a final "404 handler" ability to run arbitrary JS/TS, path match routes for dynamically running JS/TS, etc.

This can be useful for Single Page Application type setups (and more) for when you want an URL path to run certain code

The optional argument should be relative file location from the CWD that the server was run from.

For example, if you have a `public/` subdir in a repository of files that you'd like to serve static files from, and a JS file for additional "routes" in the parent dir called `routes.js`, you'd run this server like:
```sh
cd public/
deno run --allow-net --allow-read --import-map=/Users/tracey/dev/file_server_plus/import_map.json \
  /Users/tracey/dev/file_server_plus/mod.ts --cors --handler ../routes.js . # xxx update urls
```

## Full Example `routes.js`
```js
export default async function handle_or_404(req) {
  const headers = new Headers()
  try {
    const parsed = new URL(req.url)
    // main website /details/[SOMETHING] logical rewrite back to top page
    if (parsed.pathname.startsWith('/details/')) {
      headers.append('content-type', 'text/html')
      return Promise.resolve(new Response(
        Deno.readTextFileSync('./index.html'),
        { status: 200, headers },
      ))
    }
  } catch (error) {
    // some kind of error -- issue 500
    return Promise.resolve(new Response(`Server Error: ${error.message}`, { status: 500, headers }))
  }

  // not a route we want to handle -- issue 404
  headers.append('content-type', 'text/html')
  return Promise.resolve(new Response('<center><br><br><br><br>Not Found</center>', { status: 404, headers }))
}
```



## Update ourself to latest version of [file_server.ts](https://github.com/denoland/deno_std/blob/main/http/file_server.ts)
```sh
wget -qO  file_server.ts \
  https://raw.githubusercontent.com/denoland/deno_std/main/http/file_server.ts

cp  file_server.ts  mod.ts

# Previously done, after intitially made manual modifications:
#   diff -u file_server.ts mod.ts | tee plus.patch
patch  mod.ts  plus.patch

colordiff  file_server.ts  file_server_plus.ts

echo 'NOW MANUALLY UPDATE import_map.json VERSION'
```

## Compare [minimal changes](plus.patch) with stock [file_server.ts](https://github.com/denoland/deno_std/blob/main/http/file_server.ts)
```sh
colordiff  *.ts
```
