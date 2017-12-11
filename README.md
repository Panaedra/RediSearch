# RediSearch AIX 

### Full-Text search over redis by RedisLabs
![logo.png](docs/logo.png)

See the original [repo](https://github.com/RedisLabsModules/RediSearch) for code that's actually useful.

This repository fork is just for analyzing AIX portability of RediSearch. 

### What's happening as of 2017-12-11 on AIX, Redis 4.0.6, RediSearch 1.0.1

```lisp
[@srv-benchmark-aix:/src/redisearch/RediSearch-1.0.1/src]# redis-server --loadmodule /src/redisearch/RediSearch-1.0.1/src/redisearch.so
19595414:C 11 Dec 16:16:57.654 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
19595414:C 11 Dec 16:16:57.654 # Redis version=4.0.6, bits=64, commit=00000000, modified=0, pid=19595414, just started
19595414:C 11 Dec 16:16:57.654 # Configuration loaded
19595414:M 11 Dec 16:16:57.657 * Increased maximum number of open files to 10032 (it was originally set to 5000).
                _._
           _.-``__ ''-._
      _.-``    `.  `_.  ''-._           Redis 4.0.6 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 19595414
  `-._    `-._  `-./  _.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |           http://redis.io
  `-._    `-._`-.__.-'_.-'    _.-'
 |`-._`-._    `-.__.-'    _.-'_.-'|
 |    `-._`-._        _.-'_.-'    |
  `-._    `-._`-.__.-'_.-'    _.-'
      `-._    `-.__.-'    _.-'
          `-._        _.-'
              `-.__.-'

19595414:M 11 Dec 16:16:57.658 # Server initialized
19595414:M 11 Dec 16:16:57.660 * <ft> Configuration: concurrent mode: 1, ext load: , min prefix: 2, max expansions: 200,
thread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this systemthread_do(): pthread_setname_np is not supported on this system19595414:M 11 Dec 16:16:57.661 * <ft> Initialized thread pool!
19595414:M 11 Dec 16:16:57.662 * Module 'ft' loaded from /src/redisearch/RediSearch-1.0.1/src/redisearch.so
19595414:M 11 Dec 16:16:57.662 * Ready to accept connections
```

```lisp
Testing it:
[@srv-benchmark-aix:/home/src/redis-4.0.6]# redis-cli
127.0.0.1:6379> FT.CREATE "ftFoo" STOPWORDS 0 SCHEMA x TEXT
OK
127.0.0.1:6379> FT.ADD "ftFoo" "ftFoo_01" 1.0 'REPLACE' 'FIELDS' x 'hello world'
OK
127.0.0.1:6379> FT.SEARCH "ftFoo" world
1) (integer) 0
-> Nope, not working alas.
-> It did index something:
127.0.0.1:6379> keys *
1) "ft:ftFoo/world"
2) "idx:ftFoo"
3) "ft:ftFoo/hello"
4) "ftFoo_01"
127.0.0.1:6379> hgetall ftFoo_01
1) "x"
2) "hello world"

```


### Extra build steps, Notes

Last working (well, a bit):
A few loose compiles:
```lisp
gcc -Wall -Wno-unused-function -Wno-unused-variable -Wno-unused-result -fPIC -D_GNU_SOURCE -std=gnu99 -I"/src/redisearch/RediSearch-1.0.1/src" -DREDIS_MODULE_TARGET -DREDISMODULE_EXPERIMENTAL_API  -g -ggdb -O3  -maix64 -D_AIX -D_AIX53 -D_AIX61 -D_AIX71 -I/opt/freeware/include -Iinclude -c module.c -o module.o -MMD -MF module.d
gcc -Wall -Wno-unused-function -Wno-unused-variable -Wno-unused-result -fPIC -D_GNU_SOURCE -std=gnu99 -I"/src/redisearch/RediSearch-1.0.1/src" -DREDIS_MODULE_TARGET -DREDISMODULE_EXPERIMENTAL_API  -g -ggdb -O3  -maix64 -D_AIX -D_AIX53 -D_AIX61 -D_AIX71 -I/opt/freeware/include -Iinclude -c thpool.c -o thpool.o -MMD -MF thpool.d
gcc -Wall -Wno-unused-function -Wno-unused-variable -Wno-unused-result -fPIC -D_GNU_SOURCE -std=gnu99 -I"/src/redisearch/RediSearch-1.0.1/src" -DREDIS_MODULE_TARGET -DREDISMODULE_EXPERIMENTAL_API  -g -ggdb -O3  -maix64 -D_AIX -D_AIX53 -D_AIX61 -D_AIX71 -I/opt/freeware/include -Iinclude -c cndict_data.c -o cndict_data.o -MMD -MF cndict_data.d
gcc -Wall -Wno-unused-function -Wno-unused-variable -Wno-unused-result -fPIC -D_GNU_SOURCE -std=gnu99 -I"/src/redisearch/RediSearch-1.0.1/src" -DREDIS_MODULE_TARGET -DREDISMODULE_EXPERIMENTAL_API  -g -ggdb -O3  -maix64 -D_AIX -D_AIX53 -D_AIX61 -D_AIX71 -I/opt/freeware/include -Iinclude -c extension.c -o extension.o -MMD -MF extension.d
gcc -Wall -Wno-unused-function -Wno-unused-variable -Wno-unused-result -fPIC -D_GNU_SOURCE -std=gnu99 -I"/src/redisearch/RediSearch-1.0.1/src" -DREDIS_MODULE_TARGET -DREDISMODULE_EXPERIMENTAL_API  -g -ggdb -O3  -maix64 -D_AIX -D_AIX53 -D_AIX61 -D_AIX71 -I/opt/freeware/include -Iinclude -c fragmenter.c -o fragmenter.o -MMD -MF fragmenter.d
```

Linking (command made via linux output of make V=1)
```lisp
gcc -maix64 -o redisearch.so highlight_processor.o  search_request.o inverted_index.o buffer.o tokenize.o spec.o tokenize_cn.o fragmenter.o index_result.o id_list.o config.o extension.o summarize_spec.o result_processor.o id_filter.o indexer.o numeric_index.o module.o document_basic.o stemmer.o doc_table.o index.o varint.o cndict_loader.o document.o query.o numeric_filter.o byte_offsets.o redis_index.o stopwords.o tag_index.o print_version.o sortable.o geo_index.o offset_vector.o gc.o qint.o concurrent_ctx.o forward_index.o query_parser/lexer.o query_parser/parser.o ext/default.o util/heap.o util/logging.o util/mempool.o util/minmax_heap.o util/fnv.o util/khtable.o util/array.o util/block_alloc.o trie/levenshtein.o trie/rune_util.o trie/sparse_vector.o trie/trie_type.o trie/trie.o dep/thpool/thpool.o dep/cndict/cndict_data.o trie/libtrie.a dep/triemap/libtriemap.a rmutil/librmutil.a dep/libnu/libnu.a dep/friso/libfriso.a dep/snowball/libstemmer.o dep/miniz/libminiz.a -shared -Bsymbolic -Bsymbolic-functions -ldl -lpthread -maix64 -L/opt/freeware/lib64 -L/opt/freeware/lib -Wl,-blibpath:/opt/freeware/lib64:/opt/freeware/lib/pthread/ppc64:/opt/freeware/lib:/usr/lib:/lib,-bmaxdata:0x80000000 -lc -lm
```


Linking output:
```lisp
ld: 0711-224 WARNING: Duplicate symbol: LOGGING_LEVEL
ld: 0711-345 Use the -bloadmap or -bnoquiet option to obtain more information.
COLLECT_GCC_OPTIONS='-v' '-o' 'redisearch.so' '-shared' '-B' 'symbolic' '-B' 'symbolic-functions' '-maix64' '-L/opt/freeware/lib64' '-L/opt/freeware/lib'
```
-> OK, at least we've got a .so



