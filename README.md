# file_server_plus
This is a `deno` static file webserver, copied from: [file_server.ts](https://github.com/denoland/deno_std/blob/main/http/file_server.ts), with an additional final "404 handler" ability to run arbitrary JS/TS, path match routes for dynamically running JS/TS, etc.

This can be useful for Single Page Application type setups (and more) for when you want an URL path to run certain code.

The script should be relative from the static files directory.

For example, if you have a `public/` subdir in a repository of files that you'd like to serve static files from, and a JS file for additional "routes" in the parent dir called `routes.js`, you'd run this server like:
```sh
cd public/
../routes.js
```

## Full Example `routes.js`
```js
#!/usr/bin/env -S deno run --allow-net --allow-read

import main from 'https://deno.land/x/file_server_plus/mod.ts'

// Any request about to 404 arrives here and you get final ability to handle/run code or 404
globalThis.finalHandler = (req) => {
  const headers = new Headers()
  try {
    const parsed = new URL(req.url)
    if (parsed.pathname.startsWith('/details/')) {
      // main website /details/[SOMETHING] logical rewrite back to top page
      headers.append('content-type', 'text/html')
      return Promise.resolve(new Response(
        Deno.readTextFileSync('./index.html'),
        { status: 200, headers },
      ))
    }
  } catch (error) {
    // some kind of error -- issue 500
    console.warn(error)
    return Promise.resolve(new Response(`Server Error: ${error.message}`, { status: 500, headers }))
  }

  // not a route we want to handle -- issue 404
  headers.append('content-type', 'text/html')
  return Promise.resolve(new Response('<center><br><br><br><br>Not Found</center>', { status: 404, headers }))
}

main()
```


## Compare [minimal changes](plus.patch) with stock [file_server.ts](https://github.com/denoland/deno_std/blob/main/http/file_server.ts)
```sh
colordiff  *.ts
```


## Author's Note: upgrade this package to latest version of [file_server.ts](https://github.com/denoland/deno_std/blob/main/http/file_server.ts)
```sh
# get latest version from mothership
wget -qO  file_server.ts \
  https://raw.githubusercontent.com/denoland/deno_std/main/http/file_server.ts

cp  file_server.ts  mod.ts

# Re-add our small modification patch into the latest stock file_server.ts.
# ( NOTE: previously did, after intitially made manual modifications: )
#     diff -u file_server.ts mod.ts | tee plus.patch
patch  mod.ts  plus.patch

# update these both as appropriate:
VERSION_STD=0.155.0
VERSION=0.2.1

# change `from "./`  strings to `from "https://deno.land/std@$VERSION/http/`
# change `from "../` strings to `from "https://deno.land/std@$VERSION/`
perl -i -pe "s=from \x22./=from \x22https://deno.land/std\@$VERSION_STD/http/=" mod.ts
perl -i -pe "s=from \x22../=from \x22https://deno.land/std\@$VERSION_STD/=" mod.ts

colordiff -U5 *.ts


git commit -m 'updated to latest file_server.ts' .
git push
git tag         $VERSION
git push origin $VERSION

```

