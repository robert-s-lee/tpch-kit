tpch-kit
========

TPC-H benchmark kit with some modifications/additions

- Official TPC-H bechmark - [http://www.tpc.org/tpch](http://www.tpc.org/tpch)
- Original forked version - [https://github.com/gregrahn/tpch-kit](https://github.com/gregrahn/tpch-kit)

## Modifications

This fork has enhancements to support Postgres COPY ... FROM STDIN, which allows data to be loaded directly without materializeing the CSV file.
The changes are as follows:

- The default delimiter is '\t' instead of '|' whihc is default for Postgres.  
- -w command line argument specifies an alternate delimiter
- -Z command line argument specifies header before data streaming to STDOUT
- Enable EOL_HANDLING by default in the compile

# Build from source

- Linux
```bash
make -f Makefile.linux
tar -cvf - dbgen qgen dists.dss | gzip > dbgen.linux.tgz
```

- OSX
```bash
make -f Makefile.osx
tar -cvf - dbgen qgen dists.dss | gzip > dbgen.osx.tgz
```

- Windows/Cygwin
```bash
make -f Makefile.cygwin
tar -cvf - dbgen qgen dists.dss | gzip > dbgen.cygwin.tgz
```

# Installation

- OSX
```bash
cd ~/bin
gzip -dc dbgen.osx.tgz | tar -xvf -
PATH=~/bin:$PATH;export PATH
DIST=~/bin
```

- Linux
```bash
cd ~/bin
gzip -dc dbgen.linux.tgz | tar -xvf -
PATH=~/bin:$PATH;export PATH
DIST=~/bin
```

- Windows/Cygwin
```bash
cd ~/bin
gzip -dc dbgen.cygwin.tgz | tar -xvf -
PATH=~/bin:$PATH;export PATH
DIST=~/bin
```

# Exmaple loading data without materializing CSV file

- Data Size

SF=1 means Scale Factor 1.  Scale Factor can be 5,10,20,.....

These static tables don't change with SF factor

|Table  | Count |
| ----  | -----:|
|REGION |5|
|NATION |25|

The follow dynamic tables grow at SF*below row count.  At SF=1

|Table    | Count |
| ----    | -----:|
|PART     |200,000|
|SUPPLIER |10,000|
|PARTSUPP |80,0000|
|CUSTOMER |150,000|
|LINEITEM |600,0000|
|ORDERS   |1,500,0000|

- Static tables load only once

```bash
dbgen -b $DIST/dists.dss -z -Z "COPY region FROM STDIN;" -T r | psql "$DBOPT"
dbgen -b $DIST/dists.dss -z -Z "COPY nation FROM STDIN;" -T n | psql "$DBOPT"
```

- Scalable tables that scale with SF

```bash
dbgen -b $DIST/dists.dss -z -Z "COPY supplier from STDIN;" -T s | psql "$DBOPT"
dbgen -b $DIST/dists.dss -z -Z "COPY customer from STDIN;" -T c | psql "$DBOPT"
dbgen -b $DIST/dists.dss -z -Z "COPY partsupp from STDIN;" -T S | psql "$DBOPT"
dbgen -b $DIST/dists.dss -z -Z "COPY part from STDIN;" -T P | psql "$DBOPT"
```

- Scalable tables Orders and Lineitem adjusted to load at 10% of SF1 data 


Below commands breaks the data set into 10 (-C 10) and load just 1 segment (-S 1)

```bash
dbgen -b $DIST/dists.dss -z -Z "COPY orders from STDIN;" -T O -C 10 -S 1 | psql "$DBOPT"
dbgen -b $DIST/dists.dss -z -Z "COPY lineitem from STDIN;" -T L -C 10 -S 1 | psql "$DBOPT"
```

- Full orders and lineitem

Below commands loads full SF=1

```bash
dbgen -b $DIST/dists.dss -z -Z "COPY orders from STDIN;" -T O | psql "$DBOPT"
dbgen -b $DIST/dists.dss -z -Z "COPY lineitem from STDIN;" -T L | psql "$DBOPT"
```



