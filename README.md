# file_server_plus
This is a `deno` static file webserver, copied from: [file_server.ts](https://github.com/denoland/deno_std/blob/main/http/file_server.ts), with an additional optional command line arg: `--handler FILE.ts` to have a final "404 handler" ability to run arbitrary JS/TS, path match routes for dynamically running JS/TS, etc.


## Update ourself to latest version of [file_server.ts](https://github.com/denoland/deno_std/blob/main/http/file_server.ts)
```sh
wget -qO  file_server.ts  https://raw.githubusercontent.com/denoland/deno_std/main/http/file_server.ts

cp  file_server.ts  mod.ts

# Previously done, after intitially made manual modifications:
#   diff -u file_server.ts mod.ts | tee plus.patch
patch  mod.ts  plus.patch

colordiff  file_server.ts  file_server_plus.ts
```

## Compare with stock [file_server.ts](https://github.com/denoland/deno_std/blob/main/http/file_server.ts)
```sh
colordiff  *.ts
```
