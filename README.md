libmapstore
=============

Store data in memory mapped files

## Build

```bash
./autogen.sh
./configure
make
```

To run tests:
```bash
./test/tests
```

To run command line utility:
```bash
./src/mapstore --help
```

And to install locally:
```
sudo make install
```

### Debian / Ubuntu (16.04) Dependencies:

Development tools:
```bash
apt-get install build-essential libtool autotools-dev automake
```

Dependencies:
```bash
apt-get install libjson-c-dev libsqlite3-dev libuv1-dev
```

### OS X Dependencies (w/ homebrew):

Development tools:
```bash
brew install libtool automake pkgconfig
```

Dependencies:
```bash
brew install json-c sqlite libuv
```

## API

### FUNCTIONS

#### Initialize the Mapstore CTX at application start

```C
int initialize_mapstore(mapstore_ctx *ctx, mapstore_opts opts);
```

Example:
```C
  mapstore_ctx ctx;
  mapstore_opts opts;

  opts.allocation_size = 10737418240; // 10GB
  opts.map_size = 2147483648;         // 2GB
  opts.path = "~/.store";

  if (initialize_mapstore(&ctx, opts) != 0) {
      printf("Error initializing mapstore\n");
      return 1;
  }
```

#### Store Data

```C
int store_data(mapstore_ctx *ctx, int fd, uint8_t *hash);
```

#### Retrieve Data

```C
int retrieve_data(mapstore_ctx *ctx, uint8_t *hash);
```

#### Delete Data

```C
int delete_data(mapstore_ctx *ctx, uint8_t *hash);
```

#### Get Map Store Metadata

```C
store_info *get_store_info(mapstore_ctx *ctx);
```

#### Get Stored Data's Metadata
```C
data_info *get_data_info(mapstore_ctx *ctx, uint8_t *hash);
```

### STRUCTS

```C
typedef struct  {
  uint64_t allocation_size;
  uint64_t map_size;
  char *mapstore_path;
  char *database_path;
} mapstore_ctx;

typedef struct  {
  uint64_t allocation_size;
  uint64_t map_size;
  char *path;
} mapstore_opts;

typedef struct  {
  uint8_t *hash;
  char *data_locations_array;
} data_info;

typedef struct  {
  uint64_t free_space;
  uint64_t used_space;
  uint64_t allocation_size;
  uint64_t map_size;
} store_info;
```

## Architecture

TODO: Add graphical example of architecture

### SQLite Database

#### Data location table:

```
-----------------------------------------------------------------
| name | id  | data_hash | data_size | data_positions   | date  |
-----------------------------------------------------------------
| type | int | bytes(20) | int64     | Stringified JSON | int64 |
-----------------------------------------------------------------
```

`data_positions` example:

```JSON
{ shard_piece_index: [file_table_id, start_pos, end_pos], ... }
{ 0: [0,0,9], 1: [1,10,51] }
```

- 1st piece of shard is in file with id `0` from positions `0 to 9`
- 2nd piece of shard is in file with id `1` from positions `10 to 51`

#### File table:

```
----------------------------------------------
| name | id  | free_locations   | free_space |
----------------------------------------------
| type | int | Stringified JSON | int64      |
----------------------------------------------
```

free_locations example:

```JSON
{ [ [start_pos, end_pos], ... ] }
{ [ [0,9], [45,56], [51,51] ] }
```
