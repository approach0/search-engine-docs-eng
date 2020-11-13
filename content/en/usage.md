## Usage
The commands generated from building are distributed in the `run` directories of most modules.
And a lot of commands are named `test-*.out` (such as `blob-index/run/test-blob-index.out`), they are for testing purpose.
Other commands are utility or exporting commands to be used by search engine users (e.g. `searchd/run/searchd.out`).

Here we only list a few commands that are considered important
commands and they offer useful functionalities.

In general, you can issue `command -h` in most important commands to see its command line options and usage description.

### Tex parser
Run our TeX parser to see the corresponding operator tree of a math expression. And often this command is used to investigate a TeX grammar parsing error in the indexing process described later.

Below is an example of parsing \\(\dfrac a b + c\\).
```
$ ./tex-parser/run/test-tex-parser.out
edit: \frac a b +c
return string: no error (max path ID = 3).
return code: 0
Operator tree:
     └──(plus) #5, token=ADD, subtr_hash=`33891', pos=[6, 12].
           │──(frac) #4, token=FRAC, subtr_hash=`3275', pos=[6, 9].
           │     │──#1[normal`a'] #1, token=VAR, subtr_hash=`a', pos=[6, 7].
           │     └──#2[normal`b'] #2, token=VAR, subtr_hash=`b', pos=[8, 9].
           └──[normal`c'] #3, token=VAR, subtr_hash=`c', pos=[11, 12].

Suffix paths (leaf-root paths/total = 3/4):
- [path#1, leaf#1] normal`a': VAR(#1)/rank1(#0)/FRAC(#4)/ADD(#5)
* [path#1, leaf#4] 0f63: FRAC(#4)/ADD(#5) 
- [path#2, leaf#2] normal`b': VAR(#2)/rank2(#0)/FRAC(#4)/ADD(#5)
- [path#3, leaf#3] normal`c': VAR(#3)/ADD(#5) (fingerprint 0005)
```
Type `\` followed by `Tab` to auto-complete some frequently used TeX commands

### Crawler
A Python script crawler (`demo/crawler/crawler-math.stackexchange.com.py`) is included specifically for crawling *math stackexchange*.
Users are asked to write their own crawlers if they are trying to crawl data from other websites.

Install BeautifulSoup4 used by demo crawler.
```
$ apt-get install python3-pip
$ pip3 install BeautifulSoup4
```

Debian users may also need to install pycurl:
```
$ apt-get install python3-pycurl
```

To crawl *math stackexchange* from page 1 to 3:
```
$ cd $PROJECT/demo/crawler
$ ./crawler-math.stackexchange.com.py --begin-page 1 --end-page 3
```
Crawler will output all harvest files (in JSON) to `./tmp` directory which is a conventional directory name for output and will be deleted if you issue `make clean`.

You can press Ctrl-C to stop crawler in the middle of crawling process.

The output of crawler for each post will have two files, one is `*.json` corpus file (for now it contains URL and plain text of the post extracted by crawler), another is `*.html` file, which is for previewing this post corpus. (to preview it, connect to Internet and open it with your browser)

As an option, you can skip the time-consuming crawling
process by directly
[downloading a small size corpus](https://www.cs.rit.edu/~dprl/data/mse-corpus.tar.gz) (~ 930 MB) to play around later using indexer.
This small corpus contains 1000 pages we previously
crawled from *math stackexchange*.

Another crawler script `crawler-artofproblemsolving.com.py` is available for crawling [artofproblemsolving.com](https://artofproblemsolving.com). Shout out to [@TheSil](https://github.com/TheSil) for contributing that! More crawler scripts are coming out, see our plan [here](TODO.html#consider-additional-indexing-sources).

### Indexer
After corpus is generated by crawler.
Indexer is used to index a single corpus file or a corpus/collection directory recursively.

```
$ cd $PROJECT/indexer
$ ./run/indexer.out -p ./test-corpus 2> error.log
```

`test-corpus` is a test corpus manually written for specific
testing purpose.
To index the corpus you have just generated by invoking
crawler, issue:

```
$ ./run/indexer.out -p ../demo/crawler/tmp/ 2> error.log
```

Again, the output (resulting index) is generated under
`./tmp` directory except when you specify `-o` option to name
a output directory.

If you are using indexer to add new documents into existing
index in multiple runs, you need to ensure that the newly added
documents are not previously indexed. Otherwise duplicate document
may occur in search results. (Current indexer does not support index
document update)

If you are indexing a corpus with Chinese words, use `-d`
option to specify CppJieba dictionary path when calling
`indexer.out`. This will slow down indexing but it enables
searcher/searchd to search Chinese terms later (also have to
to specify `-d` in searcher/searchd).

Note it is required to have typically at least 1 GB of memory
for our indexer to successfully run through a non-trivial size
of corpus without being killed by the OS.

### Indexd
`indexd` is the daemon version of indexer, example run commmand:
```sh
$ ./run/indexd.out -o ~/nvme0n1/mnt-mathtext.img/ > /dev/null 2> error.log
```
Like indexer, `-o` option specifies the output directory.

Script `indexer/scripts/json-feeder.py` is provided to feed json files under
some directory recursively to a running indexd. Show usage from `--help` option:
```sh
$ ./scripts/json-feeder.py --help 
usage: json-feeder.py [-h] [--maxfiles MAXFILES] [--corpus-path CORPUS_PATH]
                      [--indexd-url INDEXD_URL]

Approach0 indexd json feeder.

optional arguments:
  -h, --help            show this help message and exit
  --maxfiles MAXFILES   limit the max number of files to be indexed.
  --corpus-path CORPUS_PATH
                        corpus path.
  --indexd-url INDEXD_URL
                        indexd URL (optional).
```

### Single query searcher
To test and run a query on the index you have just created,
run a *single-query searcher* that takes your query, searches for
relevant documents and exits immediately.

There are three single-query search programs available:

* A transitional full-text searcher which only searches
**terms** (i.e. regular text without math-expressions), 
located at `search/run/test-term-search.out`.
* A math-only searcher which only searches math expressions,
located at `search/run/test-math-expr-search.out`
* A mixed-query searcher which handles both math-expression
and term queries, located at `search/run/test-search.out`

Given mixed-query searcher as an example, to run mixed-query searcher
with a test query "function" and TeX "\\(f(x) = x^2 + 1\\)" on index
`../indexer/tmp`, issue:

```sh
$ cd $PROJECT/search
$ ./run/test-search.out -i ../indexer/tmp -t 'function' -m 'f(x) = x^2 + 1'
```

This searcher returns the first "page" of top-K relevant search
results (relevant keywords are highlighted in console). You
can use `-p` option to specify another page to be returned.

Please refer to command `-h` output for how to use the other
two searcher commands.

### Search daemon
On the top of our search engine modules is search daemon
program `searchd`, located at `searchd/run/searchd.out`.
It runs as a HTTP daemon that deals with every query (in JSON)
sent to it and return search results (in JSON too), never
exits unless you hit Ctrl-C.

The whole point of daemonize search service is for efficiency.
This is very easy to understand because there are obviously
large overheads in loading dictionary, and setting up
necessary search environment. Among those, the most significant
thing is caching.

Indeed, our searchd can be specified to cache a portion of
index posting into memory (currently only term index, future
will support math index too).
You can specify the maximum cache limit using `-c` option
followed by a number (in MB).
Current default cache limit is just 32MB.

To run searchd,
```
$ cd $PROJECT/searchd
$ ./run/searchd.out -i <index path> &
```

You can then test searchd by running *curl* scripts existing
in searchd directory:

```
$ ./scripts/test-query.sh ./tests/query-valid.json
```

To shutdown searchd, type command
```
$ kill -INT <pid>
```

### Search daemon cluster
Search deamon can scale to multiple nodes across multiple cores or machines.
This functionality is implemented using OpenMPI. To run two instances on single
machine, you need to copy index images to avoid index corruption. The search
results from all nodes are merged and returned from master node. Scaling can be
used to reduce search latency by searching on multiple smaller segments of
original indices.

Also, as each instance produces its own log files, it is highly recommanded to run binaries in
different folders, one can do this by simply creating two folders and symbolic
binaries in each of them.

One example command to run 3 instances over two single machines, with local machine runing 2
instances and a remote host runing 1 instance:
```
$ mpirun --host localhost,localhost,192.168.210.5 \
     -n 1 --wdir ./run1 searchd.out -i ../mnt-demo.img/ -c 800 : \
     -n 1 --wdir ./run2 searchd.out -i ../mnt-demo2.img/ -c 800 : \
     -n 1 --wdir /root ./searchd.out -i ./mnt-demo3.img/ -c 1024
```

If you are using a different SSH port for remote host, remember to edit `/etc/ssh/ssh_config` and configure ssh client:
```
Host 192.168.210.5
        Port 8985
```

And to prevent remote host prompting for password, copy `~/.ssh/id_rsa.pub` of localhost and append to `~/.ssh/authorized_keys` in remote host.

To stop all nodes in a cluster gracefully:
```
$ killall -USR1 searchd.out
```