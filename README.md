# caddy cache

[![Build Status](https://travis-ci.org/nicolasazrak/caddy-cache.svg?branch=master)](https://travis-ci.org/nicolasazrak/caddy-cache)

**Warning: This plugin is still experimental**

This is a simple caching plugin for [caddy server](https://caddyserver.com/)

## Build

To use it you need to compile your own version of caddy with this plugin. First fetch the code

- `go get -u github.com/mholt/caddy/...`
- `go get -u github.com/nicolasazrak/caddy-cache/...`

Then update the file in `$GOPATH/src/github.com/mholt/caddy/caddy/caddymain/run.go` and import `_ "github.com/nicolasazrak/caddy-cache"`.

And finally build caddy with:

- `cd $GOPATH/src/github.com/mholt/caddy/caddy`
- `./build.bash`

This will produce the caddy binary in that same folder. For more information about how plugins work read [this doc](https://github.com/mholt/caddy/wiki/Writing-a-Plugin:-Directives). 

## Usage

Example minimal usage in `Caddyfile`

```
caddy.test {
    proxy / yourserver:5000
    cache
}
```

This will store in cache responses that specifically have a `Cache-control`, `Expires` or `Last-Modified` header set.

For more advanced usages you can use the following parameters: 

- `match_path`: Paths to cache. For example `match_path /assets` will cache all successful responses for requests that start with /assets and are not marked as private.
- `match_header`: Matches responses that have the selected headers. For example `match_header Content-Type image/png image/jpg` will cache all successful responses that with content type `image/png` OR `image/jpg`. Note that if more than one is specified, anyone that matches will make the response cacheable. 
- `path`: Path where to store the cached responses. By default it will use the operating system temp folder.
- `default_max_age`: Max-age to use for matched responses that do not have an explicit expiration. (Default: 5 minutes)
- `status_header`: Sets a header to add to the response indicating the status. It will respond with: skip, miss or hit. (Default: `X-Cache-Status`)

```
caddy.test {
    proxy / yourserver:5000
    cache {
        match_path /assets
        match_header Content-Type image/jpg image/png
        status_header X-Cache-Status
        default_max_age 15m
        path /tmp/caddy-cache
    }
}
```


## Benchmarks

Benchmark files are in `benchmark` folder. Tests were run on my Lenovo G480 with Intel i3 3220 and 8gb of ram.

Test were executed with: `ab -n 2000 -c 25 http://caddy.test:2015/file.txt`


| File Size             ||                     41kb               ||                 |    608kb                ||                |   2.6M                   ||   
| ---                   |       :----:   |    :---:    |  :---:    |         ----    |    ----      | ----      |  :----:        |   ---        |   ---      |
|                       | **Total time** | **Average** | **99%th** |  **Total time** |  **Average** | **99%th** | **Total time** |  **Average** | **99%th**  |
| Proxy to Root + cache | 0.567 seconds  |  7.091 ms   |  17ms     | 0.898 seconds   | 11.224 ms    |  31 ms    |  2.525 seconds |  31.560 ms   |  51 ms     |
| Proxy to Root         | 2.683 seconds  | 33.541 ms   |  58ms     | 6.493 seconds   | 81.157 ms    | 163 ms    | 22.095 seconds | 276.187 ms   | 826 ms     |
| Root                  | 0.833 seconds  | 10.414 ms   |  23ms     | 2.546 seconds   | 31.827 ms    |  78 ms    |  8.695 seconds | 108.685 ms   | 258 ms     |

Using Gzip: 

`ab -n 100 -c 5 -H "Accept-Encoding: gzip,deflate" http://caddy.test:2015/file.txt`

| File Size             ||                     41kb               ||                 |    608kb                 ||                 |   2.6M                   ||
| ---                   |       :----:   |    :---:    |  :---:    |         ----    |    ----       | ----      |   :----:        |   ---        |   ---      |
|                       | **Total time** | **Average** | **99%th** |  **Total time** |  **Average**  | **99%th** |  **Total time** |  **Average** | **99%th**  |
| Proxy to Root + cache | 0.035 seconds  |   1.741 ms  |   5 ms    | 0.061 seconds   |    3.047 ms   |   7 ms    |   0.123 seconds |   6.154 ms   | 12 ms      |
| Proxy to Root         | 2.914 seconds  | 145.689 ms  | 285 ms    | 73.09 seconds   | 3654.508 ms   | 5709 ms   |  314.44 seconds | 16303.978 ms | 22725 ms   |
| Root                  | 2.348 seconds  | 117.406 ms  | 172 ms    | 77.59 seconds   | 3879.899 ms   | 4813 ms   |  308.66 seconds | 15433.155 ms | 20183 ms   |




## Todo list

- [x] Support `vary` header
- [x] Add header with cache status
- [x] Stream responses while fetching them from upstream
- [x] Locking concurrent requests to the same path
- [x] File disk storage for larger objects
- [ ] Purge cache entries [#1](https://github.com/nicolasazrak/caddy-cache/issues/1)
- [ ] Serve stale content if proxy is down
- [ ] Punch hole cache
- [ ] Do conditional requests to revalidate data
- [ ] Max entries size
- [ ] Add a configuration to not use query params in cache key