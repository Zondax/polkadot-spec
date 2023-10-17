---
title: "Appendix B: Host API"
---

Description of the expected environment available for import by the Polkadot Runtime

## -sec-num- Preliminaries {#id-preliminaries-4}

The Polkadot Host API is a set of functions that the Polkadot Host exposes to Runtime to access external functions needed for various reasons, such as the Storage of the content, access and manipulation, memory allocation, and also efficiency. The encoding of each data type is specified or referenced in this section. If the encoding is not mentioned, then the default Wasm encoding is used, such as little-endian byte ordering for integers.

###### Definition -def-num- Exposed Host API {#defn-host-api-at-state}

:::definition

By $\text{RE}_{{B}}$ we refer to the API exposed by the Polkadot Host, which interacts, manipulates, and responds based on the state storage whose state is set at the end of the execution of block ${B}$.

:::

###### Definition -def-num- Runtime Pointer {#defn-runtime-pointer}

:::definition

The **Runtime pointer** type is an unsigned 32-bit integer representing a pointer to data in memory. This pointer is the primary way to exchange data of fixed/known size between the Runtime and Polkadot Host.

:::

###### Definition -def-num- Runtime Pointer Size {#defn-runtime-pointer-size}

:::definition

The **Runtime pointer-size** type is an unsigned 64-bit integer representing two consecutive integers. The least significant is **Runtime pointer** ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)). The most significant provides the size of the data in bytes. This representation is the primary way to exchange data of arbitrary/dynamic sizes between the Runtime and the Polkadot Host.

:::

###### Definition -def-num- Lexicographic ordering {#defn-lexicographic-ordering}

:::definition

**Lexicographic ordering** refers to the ascending ordering of bytes or byte arrays, such as:

$$
{\left[{0},{0},{2}\right]}<{\left[{0},{1},{1}\right]}<{\left[{1}\right]}<{\left[{1},{1},{0}\right]}<{\left[{2}\right]}<{\left[\ldots\right]}
$$

The functions are specified in each subsequent subsection for each category of those functions.

:::

## -sec-num- Storage {#sect-storage-api}

Interface for accessing the storage from within the runtime.

:::danger
As of now, the storage API should silently ignore any keys that start with the `:child_storage:default:` prefix. This applies to reading and writing. If the function expects a return value, then _None_ ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) should be returned. See [substrate issue \#12461](https://github.com/paritytech/substrate/issues/12461).
:::

###### Definition -def-num- State Version {#defn-state-version}

:::definition

The state version, ${v}$, dictates how a Merkle root should be constructed. The data structure is a varying type of the following format:

$$
{v}={\left\lbrace\begin{matrix}{0}&\text{full values}\\{1}&\text{node hashes}\end{matrix}\right.}
$$

where ${0}$ indicates that the values of the keys should be inserted into the trie directly, and ${1}$ makes use of "node hashes" when calculating the Merkle proof ([Definition -def-num-ref-](chap-state#defn-hashed-subvalue)).

:::

### -sec-num- `ext_storage_set` {#sect-storage-set}

Sets the value under a given key into storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype}

    (func $ext_storage_set_version_1
        (param $key i64) (param $value i64))

**Arguments**

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the key.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the value.

### -sec-num- `ext_storage_get` {#id-ext_storage_get}

Retrieves the value associated with the given key from storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-2}

    (func $ext_storage_get_version_1
        (param $key i64) (result i64))

**Arguments**

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the key.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) returning the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the value.

### -sec-num- `ext_storage_read` {#id-ext_storage_read}

Gets the given key from storage, placing the value into a buffer and returning the number of bytes that the entry in storage has beyond the offset.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-3}

    (func $ext_storage_read_version_1
        (param $key i64) (param $value_out i64) (param $offset i32) (result i64))

**Arguments**

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the key.

- `value_out`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the buffer to which the value will be written to. This function will never write more then the length of the buffer, even if the value’s length is bigger.

- `offset`: an u32 integer (typed as i32 due to wasm types) containing the offset beyond the value should be read from.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) pointing to a SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing an unsigned 32-bit integer representing the number of bytes left at supplied `offset`. Returns _None_ if the entry does not exist.

### -sec-num- `ext_storage_clear` {#id-ext_storage_clear}

Clears the storage of the given key and its value. Non-existent entries are silently ignored.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-4}

    (func $ext_storage_clear_version_1
        (param $key_data i64))

**Arguments**

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the key.

### -sec-num- `ext_storage_exists` {#id-ext_storage_exists}

Checks whether the given key exists in storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-5}

    (func $ext_storage_exists_version_1
        (param $key_data i64) (result i32))

**Arguments**

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the key.

- `result`: an i32 integer value equal to _1_ if the key exists or a value equal to _0_ if otherwise.

### -sec-num- `ext_storage_clear_prefix` {#id-ext_storage_clear_prefix}

Clear the storage of each key/value pair where the key starts with the given prefix.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-6}

    (func $ext_storage_clear_prefix_version_1
        (param $prefix i64))

**Arguments**

- `prefix`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the prefix.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype}

    (func $ext_storage_clear_prefix_version_2
        (param $prefix i64) (param $limit i64)
        (result i64))

**Arguments**

- `prefix`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the prefix.

- `limit`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an _Option_ type ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing an unsigned 32-bit integer indicating the limit on how many keys should be deleted. No limit is applied if this is _None_. Any keys created during the current block execution do not count toward the limit.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the following variant, ${k}$:

  $$
  {k}={\left\lbrace\begin{matrix}{0}&\rightarrow{c}\\{1}&\rightarrow{c}\end{matrix}\right.}
  $$

  where _0_ indicates that all keys of the child storage have been removed, followed by the number of removed keys, ${c}$. The variant _1_ indicates that there are remaining keys, followed by the number of removed keys.

### -sec-num- `ext_storage_append` {#id-ext_storage_append}

Append the SCALE encoded value to a SCALE encoded sequence ([Definition -def-num-ref-](id-cryptography-encoding#defn-scale-list)) at the given key. This function assumes that the existing storage item is either empty or a SCALE-encoded sequence and that the value to append is also SCALE encoded and of the same type as the items in the existing sequence.

To improve performance, this function is allowed to skip decoding the entire SCALE encoded sequence and instead can just append the new item to the end of the existing data and increment the length prefix ${\text{Enc}_{{\text{SC}}}^{{\text{Len}}}}$.

:::caution
If the storage item does not exist or is not SCALE encoded, the storage item will be set to the specified value, represented as a SCALE-encoded byte array.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-7}

    (func $ext_storage_append_version_1
        (param $key i64) (param $value i64))

**Arguments**

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the key.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the value to be appended.

### -sec-num- `ext_storage_root` {#id-ext_storage_root}

Compute the storage root.

#### -sec-num- Version 1 - Prototype {#sect-ext-storage-root-version-1}

    (func $ext_storage_root_version_1
        (result i64))

**Arguments**

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to a buffer containing the 256-bit Blake2 storage root.

#### -sec-num- Version 2 - Prototype {#sect-ext-storage-root-version-2}

    (func $ext_storage_root_version_2
        (param $version i32) (result i64))

**Arguments**

- `version`: the state version ([Definition -def-num-ref-](chap-host-api#defn-state-version)).

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the buffer containing the 256-bit Blake2 storage root.

### -sec-num- `ext_storage_changes_root` {#sect-ext-storage-changes-root}

:::info
This function is not longer used and only exists for compatibility reasons.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-8}

    (func $ext_storage_changes_root_version_1
        (param $parent_hash i64) (result i64))

**Arguments**

- `parent_hash`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded block hash.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an _Option_ type ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) that’s always _None_.

### -sec-num- `ext_storage_next_key` {#id-ext_storage_next_key}

Get the next key in storage after the given one in lexicographic order ([Definition -def-num-ref-](chap-host-api#defn-lexicographic-ordering)). The key provided to this function may or may not exist in storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-9}

    (func $ext_storage_next_key_version_1
        (param $key i64) (result i64))

**Arguments**

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the next key in lexicographic order.

### -sec-num- `ext_storage_start_transaction` {#sect-ext-storage-start-transaction}

Start a new nested transaction. This allows to either commit or roll back all changes that are made after this call. For every transaction, there must be a matching call to either `ext_storage_rollback_transaction` ([Section -sec-num-ref-](chap-host-api#sect-ext-storage-rollback-transaction)) or `ext_storage_commit_transaction` ([Section -sec-num-ref-](chap-host-api#sect-ext-storage-commit-transaction)). This is also effective for all values manipulated using the child storage API ([Section -sec-num-ref-](chap-host-api#sect-child-storage-api)). It’s legal to call this function multiple times in a row.

:::caution
This is a low-level API that is potentially dangerous as it can easily result in unbalanced transactions. Runtimes should use high-level storage abstractions.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-10}

    (func $ext_storage_start_transaction_version_1)

**Arguments**

- None.

### -sec-num- `ext_storage_rollback_transaction` {#sect-ext-storage-rollback-transaction}

Rollback the last transaction started by `ext_storage_start_transaction` ([Section -sec-num-ref-](chap-host-api#sect-ext-storage-start-transaction)). Any changes made during that transaction are discarded. It’s legal to call this function multiple times in a row.

:::caution
Panics if `ext_storage_start_transaction` ([Section -sec-num-ref-](chap-host-api#sect-ext-storage-start-transaction)) was not called.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-11}

    (func $ext_storage_rollback_transaction_version_1)

**Arguments**

- None.

### -sec-num- `ext_storage_commit_transaction` {#sect-ext-storage-commit-transaction}

Commit the last transaction started by `ext_storage_start_transaction` ([Section -sec-num-ref-](chap-host-api#sect-ext-storage-start-transaction)). Any changes made during that transaction are committed to the main state. It’s legal to call this function multiple times in a row.

:::caution
Panics if `ext_storage_start_transaction` ([Section -sec-num-ref-](chap-host-api#sect-ext-storage-start-transaction)) was not called.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-12}

    (func $ext_storage_commit_transaction_version_1)

**Arguments**

- None.

## -sec-num- Child Storage {#sect-child-storage-api}

Interface for accessing the child storage from within the runtime.

###### Definition -def-num- Child Storage {#defn-child-storage-type}

:::definition

**Child storage** key is an unprefixed location of the child trie in the main trie.

:::

### -sec-num- `ext_default_child_storage_set` {#id-ext_default_child_storage_set}

Sets the value under a given key into the child storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-13}

    (func $ext_default_child_storage_set_version_1
        (param $child_storage_key i64) (param $key i64) (param $value i64))

**Arguments**

- `child_storage_key` : a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the value.

### -sec-num- `ext_default_child_storage_get` {#id-ext_default_child_storage_get}

Retrieves the value associated with the given key from the child storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-14}

    (func $ext_default_child_storage_get_version_1
        (param $child_storage_key i64) (param $key i64) (result i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the value.

### -sec-num- `ext_default_child_storage_read` {#id-ext_default_child_storage_read}

Gets the given key from storage, placing the value into a buffer and returning the number of bytes that the entry in storage has beyond the offset.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-15}

    (func $ext_default_child_storage_read_version_1
        (param $child_storage_key i64) (param $key i64) (param $value_out i64)
        (param $offset i32) (result i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `value_out`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the buffer to which the value will be written to. This function will never write more then the length of the buffer, even if the value’s length is bigger.

- `offset`: an u32 integer (typed as i32 due to wasm types) containing the offset beyond the value should be read from.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the number of bytes written into the **value_out** buffer. Returns if the entry does not exists.

### -sec-num- `ext_default_child_storage_clear` {#id-ext_default_child_storage_clear}

Clears the storage of the given key and its value from the child storage. Non-existent entries are silently ignored.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-16}

    (func $ext_default_child_storage_clear_version_1
        (param $child_storage_key i64) (param $key i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

### -sec-num- `ext_default_child_storage_storage_kill` {#id-ext_default_child_storage_storage_kill}

Clears an entire child storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-17}

    (func $ext_default_child_storage_storage_kill_version_1
        (param $child_storage_key i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-2}

    (func $ext_default_child_storage_storage_kill_version_2
        (param $child_storage_key i64) (param $limit i64)
        (result i32))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `limit`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an _Option_ type ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing an unsigned 32-bit integer indicating the limit on how many keys should be deleted. No limit is applied if this is _None_. Any keys created during the current block execution do not count toward the limit.

- `result`: a value equal to _1_ if all the keys of the child storage have been deleted or a value equal to _0_ if there are remaining keys.

#### -sec-num- Version 3 - Prototype {#id-version-3-prototype}

    (func $ext_default_child_storage_storage_kill_version_3
        (param $child_storage_key i64) (param $limit i64)
        (result i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `limit`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an _Option_ type ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing an unsigned 32-bit integer indicating the limit on how many keys should be deleted. No limit is applied if this is _None_. Any keys created during the current block execution do not count toward the limit.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the following variant, ${k}$:

  $$
  {k}={\left\lbrace\begin{matrix}{0}&\rightarrow{c}\\{1}&\rightarrow{c}\end{matrix}\right.}
  $$

  where _0_ indicates that all keys of the child storage have been removed, followed by the number of removed keys, ${c}$. The variant _1_ indicates that there are remaining keys, followed by the number of removed keys.

### -sec-num- `ext_default_child_storage_exists` {#id-ext_default_child_storage_exists}

Checks whether the given key exists in the child storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-18}

    (func $ext_default_child_storage_exists_version_1
        (param $child_storage_key i64) (param $key i64) (result i32))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `result`: an i32 integer value equal to _1_ if the key exists or a value equal to _0_ if otherwise.

### -sec-num- `ext_default_child_storage_clear_prefix` {#id-ext_default_child_storage_clear_prefix}

Clears the child storage of each key/value pair where the key starts with the given prefix.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-19}

    (func $ext_default_child_storage_clear_prefix_version_1
        (param $child_storage_key i64) (param $prefix i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `prefix`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the prefix.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-3}

    (func $ext_default_child_storage_clear_prefix_version_2
        (param $child_storage_key i64) (param $prefix i64)
        (param $limit i64) (result i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `prefix`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the prefix.

- `limit`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an _Option_ type ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing an unsigned 32-bit integer indicating the limit on how many keys should be deleted. No limit is applied if this is _None_. Any keys created during the current block execution do not count towards the limit.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the following variant, ${k}$:

  $$
  {k}={\left\lbrace\begin{matrix}{0}&\rightarrow{c}\\{1}&\rightarrow{c}\end{matrix}\right.}
  $$

  where _0_ indicates that all keys of the child storage have been removed, followed by the number of removed keys, ${c}$. The variant _1_ indicates that there are remaining keys, followed by the number of removed keys.

### -sec-num- `ext_default_child_storage_root` {#id-ext_default_child_storage_root}

Commits all existing operations and computes the resulting child storage root.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-20}

    (func $ext_default_child_storage_root_version_1
        (param $child_storage_key i64) (result i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded storage root.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-4}

    (func $ext_default_child_storage_root_version_2
        (param $child_storage_key i64) (param $version i32)
        (result i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `version`: the state version ([Definition -def-num-ref-](chap-host-api#defn-state-version)).

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit Blake2 storage root.

### -sec-num- `ext_default_child_storage_next_key` {#id-ext_default_child_storage_next_key}

Gets the next key in storage after the given one in lexicographic order ([Definition -def-num-ref-](chap-host-api#defn-lexicographic-ordering)). The key provided to this function may or may not exist in storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-21}

    (func $ext_default_child_storage_next_key_version_1
        (param $child_storage_key i64) (param $key i64) (result i64))

**Arguments**

- `child_storage_key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the child storage key ([Definition -def-num-ref-](chap-host-api#defn-child-storage-type)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded as defined in [Definition -def-num-ref-](id-cryptography-encoding#defn-option-type) containing the next key in lexicographic order. Returns if the entry cannot be found.

## -sec-num- Crypto {#sect-crypto-api}

Interfaces for working with crypto related types from within the runtime.

###### Definition -def-num- Key Type Identifier {#defn-key-type-id}

:::definition

Cryptographic keys are stored in separate key stores based on their intended use case. The separate key stores are identified by a 4-byte ASCII **key type identifier**. The following known types are available:

###### Table -tab-num- Table of known key type identifiers {#tabl-key-type-ids}

| Id   | Description                                |
| ---- | ------------------------------------------ |
| acco | Key type for the controlling accounts      |
| babe | Key type for the Babe module               |
| gran | Key type for the Grandpa module            |
| imon | Key type for the ImOnline module           |
| audi | Key type for the AuthorityDiscovery module |
| para | Key type for the Parachain Validator Key   |
| asgn | Key type for the Parachain Assignment Key  |

:::

###### Definition -def-num- ECDSA Verify Error {#defn-ecdsa-verify-error}

:::definition

**EcdsaVerifyError** is a varying data type ([Definition -def-num-ref-](id-cryptography-encoding#defn-varrying-data-type)) that specifies the error type when using ECDSA recovery functionality. The following values are possible:

###### Table -tab-num- Table of error types in ECDSA recovery {#tabl-ecdsa-verify-error}

| Id  | Description               |
| --- | ------------------------- |
| 0   | Incorrect value of R or S |
| 1   | Incorrect value of V      |
| 2   | Invalid signature         |

:::

### -sec-num- `ext_crypto_ed25519_public_keys` {#id-ext_crypto_ed25519_public_keys}

Returns all _ed25519_ public keys for the given key identifier from the keystore.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-22}

    (func $ext_crypto_ed25519_public_keys_version_1
        (param $key_type_id i32) (result i64))

**Arguments**

- `key_type_id`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key type identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an SCALE encoded 256-bit public keys.

### -sec-num- `ext_crypto_ed25519_generate` {#id-ext_crypto_ed25519_generate}

Generates an _ed25519_ key for the given key type using an optional BIP-39 seed and stores it in the keystore.

:::caution
Panics if the key cannot be generated, such as when an invalid key type or invalid seed was provided.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-23}

    (func $ext_crypto_ed25519_generate_version_1
        (param $key_type_id i32) (param $seed i64) (result i32))

**Arguments**

- `key_type_id`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key type identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `seed`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the BIP-39 seed which must be valid UTF8.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit public key.

### -sec-num- `ext_crypto_ed25519_sign` {#id-ext_crypto_ed25519_sign}

Signs the given message with the `ed25519` key that corresponds to the given public key and key type in the keystore.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-24}

    (func $ext_crypto_ed25519_sign_version_1
        (param $key_type_id i32) (param $key i32) (param $msg i64) (result i64))

**Arguments**

- `key_type_id`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key type identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `key`: a pointer to the buffer containing the 256-bit public key.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be signed.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the 64-byte signature. This function returns if the public key cannot be found in the key store.

### -sec-num- `ext_crypto_ed25519_verify` {#sect-ext-crypto-ed25519-verify}

Verifies an _ed25519_ signature.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-25}

    (func $ext_crypto_ed25519_verify_version_1
        (param $sig i32) (param $msg i64) (param $key i32) (result i32))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 64-byte signature.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be verified.

- `key`: a pointer to the buffer containing the 256-bit public key.

- `result`: a i32 integer value equal to _1_ if the signature is valid or a value equal to _0_ if otherwise.

### -sec-num- `ext_crypto_ed25519_batch_verify` {#sect-ext-crypto-ed25519-batch-verify}

Registers an ed25519 signature for batch verification. Batch verification is enabled by calling `ext_crypto_start_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-start-batch-verify)). The result of the verification is returned by `ext_crypto_finish_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-finish-batch-verify)). If batch verification is not enabled, the signature is verified immediately.

#### -sec-num- Version 1 {#id-version-1}

    (func $ext_crypto_ed25519_batch_verify_version_1
        (param $sig i32) (param $msg i64) (param $key i32) (result i32))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 64-byte signature.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be verified.

- `key`: a pointer to the buffer containing the 256-bit public key.

- `result`: an i32 integer value equal to _1_ if the signature is valid or batched or a value equal _0_ to if otherwise.

### -sec-num- `ext_crypto_sr25519_public_keys` {#id-ext_crypto_sr25519_public_keys}

Returns all _sr25519_ public keys for the given key id from the keystore.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-26}

    (func $ext_crypto_sr25519_public_keys_version_1
        (param $key_type_id i32) (result i64))

**Arguments**

- `key_type_id`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key type identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded 256-bit public keys.

### -sec-num- `ext_crypto_sr25519_generate` {#id-ext_crypto_sr25519_generate}

Generates an _sr25519_ key for the given key type using an optional BIP-39 seed and stores it in the keystore.

:::caution
Panics if the key cannot be generated, such as when an invalid key type or invalid seed was provided.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-27}

    (func $ext_crypto_sr25519_generate_version_1
        (param $key_type_id i32) (param $seed i64) (result i32))

**Arguments**

- `key_type_id`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `seed`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the BIP-39 seed which must be valid UTF8.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit public key.

### -sec-num- `ext_crypto_sr25519_sign` {#id-ext_crypto_sr25519_sign}

Signs the given message with the _sr25519_ key that corresponds to the given public key and key type in the keystore.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-28}

    (func $ext_crypto_sr25519_sign_version_1
        (param $key_type_id i32) (param $key i32) (param $msg i64) (result i64))

**Arguments**

- `key_type_id`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `key`: a pointer to the buffer containing the 256-bit public key.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be signed.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the 64-byte signature. This function returns _None_ if the public key cannot be found in the key store.

### -sec-num- `ext_crypto_sr25519_verify` {#sect-ext-crypto-sr25519-verify}

Verifies an sr25519 signature.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-29}

    (func $ext_crypto_sr25519_verify_version_1
        (param $sig i32) (param $msg i64) (param $key i32) (result i32))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 64-byte signature.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be verified.

- `key`: a pointer to the buffer containing the 256-bit public key.

- `result`: a i32 integer value equal to _1_ if the signature is valid or a value equal to _0_ if otherwise.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-5}

    (func $ext_crypto_sr25519_verify_version_2
        (param $sig i32) (param $msg i64) (param $key i32) (result i32))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 64-byte signature.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be verified.

- `key`: a pointer to the buffer containing the 256-bit public key.

- `result`: a i32 integer value equal to _1_ if the signature is valid or a value equal to _0_ if otherwise.

### -sec-num- `ext_crypto_sr25519_batch_verify` {#sect-ext-crypto-sr25519-batch-verify}

Registers a sr25519 signature for batch verification. Batch verification is enabled by calling `ext_crypto_start_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-start-batch-verify)). The result of the verification is returned by `ext_crypto_finish_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-finish-batch-verify)). If batch verification is not enabled, the signature is verified immediately.

#### -sec-num- Version 1 {#id-version-1-2}

    (func $ext_crypto_sr25519_batch_verify_version_1
        (param $sig i32) (param $msg i64) (param $key i32) (result i32))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 64-byte signature.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be verified.

- `key`: a pointer to the buffer containing the 256-bit public key.

- `result`: an i32 integer value equal to _1_ if the signature is valid or batched or a value equal _0_ to if otherwise.

### -sec-num- `ext_crypto_ecdsa_public_keys` {#id-ext_crypto_ecdsa_public_keys}

Returns all _ecdsa_ public keys for the given key id from the keystore.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-30}

    (func $ext_crypto_ecdsa_public_key_version_1
        (param $key_type_id i64) (result i64))

**Arguments**

- `key_type_id`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key type identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded 33-byte compressed public keys.

### -sec-num- `ext_crypto_ecdsa_generate` {#id-ext_crypto_ecdsa_generate}

Generates an _ecdsa_ key for the given key type using an optional BIP-39 seed and stores it in the keystore.

:::caution
Panics if the key cannot be generated, such as when an invalid key type or invalid seed was provided.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-31}

    (func $ext_crypto_ecdsa_generate_version_1
        (param $key_type_id i32) (param $seed i64) (result i32))

**Arguments**

- `key_type_id`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `seed`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the BIP-39 seed which must be valid UTF8.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 33-byte compressed public key.

### -sec-num- `ext_crypto_ecdsa_sign` {#id-ext_crypto_ecdsa_sign}

Signs the hash of the given message with the _ecdsa_ key that corresponds to the given public key and key type in the keystore.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-32}

    (func $ext_crypto_ecdsa_sign_version_1
        (param $key_type_id i32) (param $key i32) (param $msg i64) (result i64))

**Arguments**

- `key_type_id`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `key`: a pointer to the buffer containing the 33-byte compressed public key.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be signed.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the signature. The signature is 65-bytes in size, where the first 512-bits represent the signature and the other 8 bits represent the recovery ID. This function returns if the public key cannot be found in the key store.

### -sec-num- `ext_crypto_ecdsa_sign_prehashed` {#id-ext_crypto_ecdsa_sign_prehashed}

Signs the prehashed message with the _ecdsa_ key that corresponds to the given public key and key type in the keystore.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-33}

    (func $ext_crypto_ecdsa_sign_prehashed_version_1
        (param $key_type_id i32) (param $key i32) (param $msg i64) (result i64))

**Arguments**

- `key_type_id`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the key identifier ([Definition -def-num-ref-](chap-host-api#defn-key-type-id)).

- `key`: a pointer to the buffer containing the 33-byte compressed public key.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be signed.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the signature. The signature is 65-bytes in size, where the first 512-bits represent the signature and the other 8 bits represent the recovery ID. This function returns if the public key cannot be found in the key store.

### -sec-num- `ext_crypto_ecdsa_verify` {#sect-ext-crypto-ecdsa-verify}

Verifies the hash of the given message against an ECDSA signature.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-34}

This function allows the verification of non-standard, overflowing ECDSA signatures, an implementation specific mechanism of the Rust [`libsecp256k1` library](https://github.com/paritytech/libsecp256k1), specifically the [`parse_overflowing`](https://docs.rs/libsecp256k1/0.7.0/libsecp256k1/struct.Signature.html#method.parse_overflowing) function.

    (func $ext_crypto_ecdsa_verify_version_1
        (param $sig i32) (param $msg i64) (param $key i32) (result i32))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 65-byte signature. The signature is 65-bytes in size, where the first 512-bits represent the signature and the other 8 bits represent the recovery ID.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be verified.

- `key`: a pointer to the buffer containing the 33-byte compressed public key.

- `result`: a i32 integer value equal _1_ to if the signature is valid or a value equal to _0_ if otherwise.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-6}

Does not allow the verification of non-standard, overflowing ECDSA signatures.

    (func $ext_crypto_ecdsa_verify_version_2
        (param $sig i32) (param $msg i64) (param $key i32) (result i32))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 65-byte signature. The signature is 65-bytes in size, where the first 512-bits represent the signature and the other 8 bits represent the recovery ID.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be verified.

- `key`: a pointer to the buffer containing the 33-byte compressed public key.

- `result`: a i32 integer value equal _1_ to if the signature is valid or a value equal to _0_ if otherwise.

### -sec-num- `ext_crypto_ecdsa_verify_prehashed` {#id-ext_crypto_ecdsa_verify_prehashed}

Verifies the prehashed message against a ECDSA signature.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-35}

    (func $ext_crypto_ecdsa_verify_prehashed_version_1
        (param $sig i32) (param $msg i32) (param $key i32) (result i32))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 65-byte signature. The signature is 65-bytes in size, where the first 512-bits represent the signature and the other 8 bits represent the recovery ID.

- `msg`: a pointer to the 32-bit prehashed message to be verified.

- `key`: a pointer to the 33-byte compressed public key.

- `result`: a i32 integer value equal _1_ to if the signature is valid or a value equal to _0_ if otherwise.

### -sec-num- `ext_crypto_ecdsa_batch_verify` {#sect-ext-crypto-ecdsa-batch-verify}

Registers a ECDSA signature for batch verification. Batch verification is enabled by calling `ext_crypto_start_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-start-batch-verify)). The result of the verification is returned by `ext_crypto_finish_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-finish-batch-verify)). If batch verification is not enabled, the signature is verified immediately.

#### -sec-num- Version 1 {#id-version-1-3}

    (func $ext_crypto_ecdsa_batch_verify_version_1
        (param $sig i32) (param $msg i64) (param $key i32) (result i32))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 64-byte signature.

- `msg`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the message that is to be verified.

- `key`: a pointer to the buffer containing the 256-bit public key.

- `result`: a i32 integer value equal to _1_ if the signature is valid or batched or a value equal _0_ to if otherwise.

### -sec-num- `ext_crypto_secp256k1_ecdsa_recover` {#id-ext_crypto_secp256k1_ecdsa_recover}

Verify and recover a _secp256k1_ ECDSA signature.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-36}

This function can handle non-standard, overflowing ECDSA signatures, an implemenation specific mechanism of the Rust [`libsecp256k1` library](https://github.com/paritytech/libsecp256k1), specifically the [`parse_overflowing`](https://docs.rs/libsecp256k1/0.7.0/libsecp256k1/struct.Signature.html#method.parse_overflowing) function.

    (func $ext_crypto_secp256k1_ecdsa_recover_version_1
        (param $sig i32) (param $msg i32) (result i64))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 65-byte signature in RSV format. V should be either `0/1` or `27/28`.

- `msg`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit Blake2 hash of the message.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Result_ ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)). On success it contains the 64-byte recovered public key or an error type ([Definition -def-num-ref-](chap-host-api#defn-ecdsa-verify-error)) on failure.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-7}

Does not handle non-standard, overflowing ECDSA signatures.

    (func $ext_crypto_secp256k1_ecdsa_recover_version_2
        (param $sig i32) (param $msg i32) (result i64))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 65-byte signature in RSV format. V should be either or .

- `msg`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit Blake2 hash of the message.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Result_ ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)). On success it contains the 64-byte recovered public key or an error type ([Definition -def-num-ref-](chap-host-api#defn-ecdsa-verify-error)) on failure.

### -sec-num- `ext_crypto_secp256k1_ecdsa_recover_compressed` {#id-ext_crypto_secp256k1_ecdsa_recover_compressed}

Verify and recover a _secp256k1_ ECDSA signature.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-37}

This function can handle non-standard, overflowing ECDSA signatures, an implemenation specific mechanism of the Rust [`libsecp256k1` library](https://github.com/paritytech/libsecp256k1), specifically the [`parse_overflowing`](https://docs.rs/libsecp256k1/0.7.0/libsecp256k1/struct.Signature.html#method.parse_overflowing) function.

    (func $ext_crypto_secp256k1_ecdsa_recover_compressed_version_1
        (param $sig i32) (param $msg i32) (result i64))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 65-byte signature in RSV format. V should be either `0/1` or `27/28`.

- `msg`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit Blake2 hash of the message.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded `Result` value ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)). On success it contains the 33-byte recovered public key in compressed form on success or an error type ([Definition -def-num-ref-](chap-host-api#defn-ecdsa-verify-error)) on failure.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-8}

Does not handle non-standard, overflowing ECDSA signatures.

    (func $ext_crypto_secp256k1_ecdsa_recover_compressed_version_2
        (param $sig i32) (param $msg i32) (result i64))

**Arguments**

- `sig`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 65-byte signature in RSV format. V should be either `0/1` or `27/28`.

- `msg`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit Blake2 hash of the message.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded `Result` value ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)). On success it contains the 33-byte recovered public key in compressed form on success or an error type ([Definition -def-num-ref-](chap-host-api#defn-ecdsa-verify-error)) on failure.

### -sec-num- `ext_crypto_start_batch_verify` {#sect-ext-crypto-start-batch-verify}

Starts the verification extension. The extension is a separate background process and is used to parallel-verify signatures which are pushed to the batch with `ext_crypto_ed25519_batch_verify`([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-ed25519-batch-verify)), `ext_crypto_sr25519_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-sr25519-batch-verify)) or `ext_crypto_ecdsa_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-ecdsa-batch-verify)). Verification will start immediately and the Runtime can retrieve the result when calling `ext_crypto_finish_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-finish-batch-verify)).

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-38}

    (func $ext_crypto_start_batch_verify_version_1)

**Arguments**

- None.

### -sec-num- `ext_crypto_finish_batch_verify` {#sect-ext-crypto-finish-batch-verify}

Finish verifying the batch of signatures since the last call to this function. Blocks until all the signatures are verified.

:::caution
Panics if `ext_crypto_start_batch_verify` ([Section -sec-num-ref-](chap-host-api#sect-ext-crypto-start-batch-verify)) was not called.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-39}

    (func $ext_crypto_finish_batch_verify_version_1
        (result i32))

**Arguments**

- `result`: an i32 integer value equal to _1_ if all the signatures are valid or a value equal to _0_ if one or more of the signatures are invalid.

## -sec-num- Hashing {#sect-hashing-api}

Interface that provides functions for hashing with different algorithms.

### -sec-num- `ext_hashing_keccak_256` {#id-ext_hashing_keccak_256}

Conducts a 256-bit Keccak hash.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-40}

    (func $ext_hashing_keccak_256_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the data to be hashed.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit hash result.

### -sec-num- `ext_hashing_keccak_512` {#id-ext_hashing_keccak_512}

Conducts a 512-bit Keccak hash.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-41}

    (func $ext_hashing_keccak_512_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the data to be hashed.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 512-bit hash result.

### -sec-num- `ext_hashing_sha2_256` {#id-ext_hashing_sha2_256}

Conducts a 256-bit Sha2 hash.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-42}

    (func $ext_hashing_sha2_256_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the data to be hashed.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit hash result.

### -sec-num- `ext_hashing_blake2_128` {#id-ext_hashing_blake2_128}

Conducts a 128-bit Blake2 hash.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-43}

    (func $ext_hashing_blake2_128_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the data to be hashed.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 128-bit hash result.

### -sec-num- `ext_hashing_blake2_256` {#id-ext_hashing_blake2_256}

Conducts a 256-bit Blake2 hash.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-44}

    (func $ext_hashing_blake2_256_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the data to be hashed.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit hash result.

### -sec-num- `ext_hashing_twox_64` {#id-ext_hashing_twox_64}

Conducts a 64-bit xxHash hash.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-45}

    (func $ext_hashing_twox_64_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the data to be hashed.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 64-bit hash result.

### -sec-num- `ext_hashing_twox_128` {#id-ext_hashing_twox_128}

Conducts a 128-bit xxHash hash.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-46}

    (func $ext_hashing_twox_128
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the data to be hashed.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 128-bit hash result.

### -sec-num- `ext_hashing_twox_256` {#id-ext_hashing_twox_256}

Conducts a 256-bit xxHash hash.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-47}

    (func $ext_hashing_twox_256
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the data to be hashed.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit hash result.

## -sec-num- Offchain {#sect-offchain-api}

The Offchain Workers allow the execution of long-running and possibly non-deterministic tasks (e.g. web requests, encryption/decryption and signing of data, random number generation, CPU-intensive computations, enumeration/aggregation of on-chain data, etc.) which could otherwise require longer than the block execution time. Offchain Workers have their own execution environment. This separation of concerns is to make sure that the block production is not impacted by the long-running tasks.

All data and results generated by Offchain workers are unique per node and nondeterministic. Information can be propagated to other nodes by submitting a transaction that should be included in the next block. As Offchain workers runs on their own execution environment they have access to their own separate storage. There are two different types of storage available which are defined in [Definition -def-num-ref-](chap-host-api#defn-offchain-persistent-storage) and [Definition -def-num-ref-](chap-host-api#defn-offchain-local-storage).

###### Definition -def-num- Persisted Storage {#defn-offchain-persistent-storage}

:::definition

**Persistent storage** is non-revertible and not fork-aware. It means that any value set by the offchain worker is persisted even if that block (at which the worker is called) is reverted as non-canonical (meaning that the block was surpassed by a longer chain). The value is available for the worker that is re-run at the new (different block with the same block number) and future blocks. This storage can be used by offchain workers to handle forks and coordinate offchain workers running on different forks.

:::

###### Definition -def-num- Local Storage {#defn-offchain-local-storage}

:::definition

**Local storage** is revertible and fork-aware. It means that any value set by the offchain worker triggered at a certain block is reverted if that block is reverted as non-canonical. The value is NOT available for the worker that is re-run at the next or any future blocks.

:::

###### Definition -def-num- HTTP Status Code {#defn-http-status-code}

:::definition

An enumerated data type that holds a finite set of distinct variants that gets SCALE-encoded as described in ([Definition -def-num-ref-](id-cryptography-encoding#defn-scale-variable-type)) and returned by offchain http functions. The set of variants is defined as follows.

| Id  | Name                    | Description                                               |
| --- | ----------------------- | --------------------------------------------------------- |
| 0   | _DeadlineReached_       | the deadline for the started request was reached.         |
| 1   | _IoError_               | an error has occurred during the request.                 |
| 2   | _Invalid_               | the specified request identifier is invalid.              |
| 3   | _Finished(http_status)_ | the request has finished with the given HTTP status code. |

**where**

- `http_status`: a 16-bit unsigned integer type representing the [HTTP status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status) to be returned.

:::

###### Definition -def-num- HTTP Error {#defn-http-error}

:::definition

HTTP error, ${E}$, is a varying data type ([Definition -def-num-ref-](id-cryptography-encoding#defn-varrying-data-type)) and specifies the error types of certain HTTP functions. Following values are possible:

$$
{E}={\left\lbrace\begin{matrix}{0}&\text{The deadile was reached}\\{1}&\text{There was an IO error while processing the request}\\{2}&\text{The Id of the request is invalid}\end{matrix}\right.}
$$

:::

### -sec-num- `ext_offchain_is_validator` {#id-ext_offchain_is_validator}

Check whether the local node is a potential validator. Even if this function returns _1_, it does not mean that any keys are configured or that the validator is registered in the chain.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-48}

    (func $ext_offchain_is_validator_version_1 (result i32))

**Arguments**

- `result`: a i32 integer which is equal to _1_ if the local node is a potential validator or a integer equal to _0_ if it is not.

### -sec-num- `ext_offchain_submit_transaction` {#sect-ext-offchain-submit-transaction}

Given a SCALE encoded extrinsic, this function submits the extrinsic to the Host’s transaction pool, ready to be propagated to remote peers.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-49}

    (func $ext_offchain_submit_transaction_version_1
        (param $data i64) (result i64))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the byte array storing the encoded extrinsic.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Result_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)). Neither on success or failure is there any additional data provided. The cause of a failure is implementation specific.

### -sec-num- `ext_offchain_network_state` {#id-ext_offchain_network_state}

Returns the SCALE encoded, opaque information about the local node’s network state.

###### Definition -def-num- Opaque Network State {#defn-opaque-network-state}

:::definition

The **Opaque network state structure**, ${S}$, is a SCALE encoded blob holding information about the the _libp2p PeerId_, ${P}_{{\text{id}}}$, of the local node and a list of _libp2p Multiaddresses_, ${\left({M}_{{0}},\ldots{M}_{{n}}\right)}$, the node knows it can be reached at:

$$
{S}={\left({P}_{{\text{id}}},{\left({M}_{{0}},\ldots{M}_{{n}}\right)}\right)}
$$

**where**

$$
{P}_{{\text{id}}}={\left({b}_{{0}},\ldots{b}_{{n}}\right)}
$$

$$
{M}={\left({b}_{{0}},\ldots{b}_{{n}}\right)}
$$

The information contained in this structure is naturally opaque to the caller of this function.

:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-50}

    (func $ext_offchain_network_state_version_1 (result i64))

**Arguments**

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded `Result` value ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)). On success it contains the _Opaque network state_ structure ([Definition -def-num-ref-](chap-host-api#defn-opaque-network-state)). On failure, an empty value is yielded where its cause is implementation specific.

### -sec-num- `ext_offchain_timestamp` {#id-ext_offchain_timestamp}

Returns the current timestamp.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-51}

    (func $ext_offchain_timestamp_version_1 (result i64))

**Arguments**

- `result`: an u64 integer (typed as i64 due to wasm types) indicating the current UNIX timestamp ([Definition -def-num-ref-](id-cryptography-encoding#defn-unix-time)).

### -sec-num- `ext_offchain_sleep_until` {#id-ext_offchain_sleep_until}

Pause the execution until the `deadline` is reached.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-52}

    (func $ext_offchain_sleep_until_version_1 (param $deadline i64))

**Arguments**

- `deadline`: an u64 integer (typed as i64 due to wasm types) specifying the UNIX timestamp ([Definition -def-num-ref-](id-cryptography-encoding#defn-unix-time)).

### -sec-num- `ext_offchain_random_seed` {#id-ext_offchain_random_seed}

Generates a random seed. This is a truly random non deterministic seed generated by the host environment.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-53}

    (func $ext_offchain_random_seed_version_1 (result i32))

**Arguments**

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit seed.

### -sec-num- `ext_offchain_local_storage_set` {#id-ext_offchain_local_storage_set}

Sets a value in the local storage. This storage is not part of the consensus, it’s only accessible by the offchain worker tasks running on the same machine and is persisted between runs.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-54}

    (func $ext_offchain_local_storage_set_version_1
        (param $kind i32) (param $key i64) (param $value i64))

**Arguments**

- `kind`: an i32 integer indicating the storage kind. A value equal to _1_ is used for a persistent storage ([Definition -def-num-ref-](chap-host-api#defn-offchain-persistent-storage)) and a value equal to _2_ for local storage ([Definition -def-num-ref-](chap-host-api#defn-offchain-local-storage)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the value.

### -sec-num- `ext_offchain_local_storage_clear` {#id-ext_offchain_local_storage_clear}

Remove a value from the local storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-55}

    (func $ext_offchain_local_storage_clear_version_1
        (param $kind i32) (param $key i64))

**Arguments**

- `kind`: an i32 integer indicating the storage kind. A value equal to _1_ is used for a persistent storage ([Definition -def-num-ref-](chap-host-api#defn-offchain-persistent-storage)) and a value equal to _2_ for local storage ([Definition -def-num-ref-](chap-host-api#defn-offchain-local-storage)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

### -sec-num- `ext_offchain_local_storage_compare_and_set` {#id-ext_offchain_local_storage_compare_and_set}

Sets a new value in the local storage if the condition matches the current value.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-56}

    (fund $ext_offchain_local_storage_compare_and_set_version_1
        (param $kind i32) (param $key i64) (param $old_value i64)
        (param $new_value i64) (result i32))

**Arguments**

- `kind`: an i32 integer indicating the storage kind. A value equal to _1_ is used for a persistent storage ([Definition -def-num-ref-](chap-host-api#defn-offchain-persistent-storage)) and a value equal to _2_ for local storage ([Definition -def-num-ref-](chap-host-api#defn-offchain-local-storage)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `old_value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the old key.

- `new_value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the new value.

- `result`: an i32 integer equal to _1_ if the new value has been set or a value equal to _0_ if otherwise.

### -sec-num- `ext_offchain_local_storage_get` {#id-ext_offchain_local_storage_get}

Gets a value from the local storage.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-57}

    (func $ext_offchain_local_storage_get_version_1
        (param $kind i32) (param $key i64) (result i64))

**Arguments**

- `kind`: an i32 integer indicating the storage kind. A value equal to _1_ is used for a persistent storage ([Definition -def-num-ref-](chap-host-api#defn-offchain-persistent-storage)) and a value equal to _2_ for local storage ([Definition -def-num-ref-](chap-host-api#defn-offchain-local-storage)).

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the value or the corresponding key.

### -sec-num- `ext_offchain_http_request_start` {#id-ext_offchain_http_request_start}

Initiates a HTTP request given by the HTTP method and the URL. Returns the Id of a newly started request.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-58}

    (func $ext_offchain_http_request_start_version_1
      (param $method i64) (param $uri i64) (param $meta i64) (result i64))

**Arguments**

- `method`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the HTTP method. Possible values are “GET” and “POST”.

- `uri`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the URI.

- `meta`: a future-reserved field containing additional, SCALE encoded parameters. Currently, an empty array should be passed.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Result_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)) containing the i16 ID of the newly started request. On failure no additionally data is provided. The cause of failure is implementation specific.

### -sec-num- `ext_offchain_http_request_add_header` {#id-ext_offchain_http_request_add_header}

Append header to the request. Returns an error if the request identifier is invalid, `http_response_wait` has already been called on the specified request identifier, the deadline is reached or an I/O error has happened (e.g. the remote has closed the connection).

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-59}

    (func $ext_offchain_http_request_add_header_version_1
        (param $request_id i32) (param $name i64) (param $value i64) (result i64))

**Arguments**

- `request_id`: an i32 integer indicating the ID of the started request.

- `name`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the HTTP header name.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the HTTP header value.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Result_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)). Neither on success or failure is there any additional data provided. The cause of failure is implementation specific.

### -sec-num- `ext_offchain_http_request_write_body` {#id-ext_offchain_http_request_write_body}

Writes a chunk of the request body. Returns a non-zero value in case the deadline is reached or the chunk could not be written.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-60}

    (func $ext_offchain_http_request_write_body_version_1
        (param $request_id i32) (param $chunk i64) (param $deadline i64) (result i64))

**Arguments**

- `request_id`: an i32 integer indicating the ID of the started request.

- `chunk`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the chunk of bytes. Writing an empty chunk finalizes the request.

- `deadline`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the UNIX timestamp ([Definition -def-num-ref-](id-cryptography-encoding#defn-unix-time)). Passing _None_ blocks indefinitely.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Result_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)). On success, no additional data is provided. On error it contains the HTTP error type ([Definition -def-num-ref-](chap-host-api#defn-http-error)).

### -sec-num- `ext_offchain_http_response_wait` {#id-ext_offchain_http_response_wait}

Returns an array of request statuses (the length is the same as IDs). Note that if deadline is not provided the method will block indefinitely, otherwise unready responses will produce DeadlineReached status.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-61}

    (func $ext_offchain_http_response_wait_version_1
        (param $ids i64) (param $deadline i64) (result i64))

**Arguments**

- `ids`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded array of started request IDs.

- `deadline`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the UNIX timestamp ([Definition -def-num-ref-](id-cryptography-encoding#defn-unix-time)). Passing None blocks indefinitely.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded array of request statuses ([Definition -def-num-ref-](chap-host-api#defn-http-status-code)).

### -sec-num- `ext_offchain_http_response_headers` {#id-ext_offchain_http_response_headers}

Read all HTTP response headers. Returns an array of key/value pairs. Response headers must be read before the response body.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-62}

    (func $ext_offchain_http_response_headers_version_1
        (param $request_id i32) (result i64))

**Arguments**

- `request_id`: an i32 integer indicating the ID of the started request.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to a SCALE encoded array of key/value pairs.

### -sec-num- `ext_offchain_http_response_read_body` {#id-ext_offchain_http_response_read_body}

Reads a chunk of body response to the given buffer. Returns the number of bytes written or an error in case a deadline is reached or the server closed the connection. If 0 is returned it means that the response has been fully consumed and the request_id is now invalid. This implies that response headers must be read before draining the body.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-63}

    (func $ext_offchain_http_response_read_body_version_1
        (param $request_id i32) (param $buffer i64) (param $deadline i64) (result i64))

**Arguments**

- `request_id`: an i32 integer indicating the ID of the started request.

- `buffer`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the buffer where the body gets written to.

- `deadline`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the UNIX timestamp ([Definition -def-num-ref-](id-cryptography-encoding#defn-unix-time)). Passing _None_ will block indefinitely.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Result_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-result-type)). On success it contains an i32 integer specifying the number of bytes written or a HTTP error type ([Definition -def-num-ref-](chap-host-api#defn-http-error)) on failure.

## -sec-num- Offchain Index {#sect-offchainindex-api}

Interface that provides functions to access the Offchain DB through offchain indexing.

### -sec-num- `Offchain_index_set` {#id-offchain_index_set}

Write a key-value pair to the Offchain DB in a buffered fashion.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-Offchain_index_set}

    (func $ext_offchain_index_set_version_1
        (param $key i64) (param $value i64))

**Arguments**

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the key.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the value.

### -sec-num- `Offchain_index_clear` {#id-offchain_index_clear}

Remove a key and its associated value from the Offchain DB.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-Offchain_index_clear}

    (func $ext_offchain_index_set_version_1
        (param $key i64))

**Arguments**

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) containing the key.

## -sec-num- Trie {#sect-trie-api}

Interface that provides trie related functionality.

### -sec-num- `ext_trie_blake2_256_root` {#id-ext_trie_blake2_256_root}

Compute a 256-bit Blake2 trie root formed from the iterated items.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-64}

    (func $ext_trie_blake2_256_root_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the iterated items from which the trie root gets formed. The items consist of a SCALE encoded array containing arbitrary key/value pairs (tuples).

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit trie root.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-9}

    (func $ext_trie_blake2_256_root_version_2
        (param $data i64) (param $version i32)
        (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the iterated items from which the trie root gets formed. The items consist of a SCALE encoded array containing arbitrary key/value pairs (tuples).

- `version`: the state version ([Definition -def-num-ref-](chap-host-api#defn-state-version)).

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit trie root.

### -sec-num- `ext_trie_blake2_256_ordered_root` {#id-ext_trie_blake2_256_ordered_root}

Compute a 256-bit Blake2 trie root formed from the enumerated items.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-65}

    (func $ext_trie_blake2_256_ordered_root_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the enumerated items from which the trie root gets formed. The items consist of a SCALE encoded array containing only values, where the corresponding key of each value is the index of the item in the array, starting at 0. The keys are compact encoded integers ([Definition -def-num-ref-](id-cryptography-encoding#defn-sc-len-encoding)).

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit trie root result.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-10}

    (func $ext_trie_blake2_256_ordered_root_version_2
        (param $data i64) (param $version i32)
        (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the enumerated items from which the trie root gets formed. The items consist of a SCALE encoded array containing only values, where the corresponding key of each value is the index of the item in the array, starting at 0. The keys are compact encoded integers ([Definition -def-num-ref-](id-cryptography-encoding#defn-sc-len-encoding)).

- `version`: the state version ([Definition -def-num-ref-](chap-host-api#defn-state-version)).

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit trie root result.

### -sec-num- `ext_trie_keccak_256_root` {#id-ext_trie_keccak_256_root}

Compute a 256-bit Keccak trie root formed from the iterated items.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-66}

    (func $ext_trie_keccak_256_root_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the iterated items from which the trie root gets formed. The items consist of a SCALE encoded array containing arbitrary key/value pairs.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit trie root.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-11}

    (func $ext_trie_keccak_256_root_version_2
        (param $data i64) (param $version i32)
        (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the iterated items from which the trie root gets formed. The items consist of a SCALE encoded array containing arbitrary key/value pairs.

- `version`: the state version ([Definition -def-num-ref-](chap-host-api#defn-state-version)).

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit trie root.

### -sec-num- `ext_trie_keccak_256_ordered_root` {#id-ext_trie_keccak_256_ordered_root}

Compute a 256-bit Keccak trie root formed from the enumerated items.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-67}

    (func $ext_trie_keccak_256_ordered_root_version_1
        (param $data i64) (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the enumerated items from which the trie root gets formed. The items consist of a SCALE encoded array containing only values, where the corresponding key of each value is the index of the item in the array, starting at 0. The keys are compact encoded integers ([Definition -def-num-ref-](id-cryptography-encoding#defn-sc-len-encoding)).

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit trie root result.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-12}

    (func $ext_trie_keccak_256_ordered_root_version_2
        (param $data i64) (param $version i32)
        (result i32))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the enumerated items from which the trie root gets formed. The items consist of a SCALE encoded array containing only values, where the corresponding key of each value is the index of the item in the array, starting at 0. The keys are compact encoded integers ([Definition -def-num-ref-](id-cryptography-encoding#defn-sc-len-encoding)).

- `version`: the state version ([Definition -def-num-ref-](chap-host-api#defn-state-version)).

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the buffer containing the 256-bit trie root result.

### -sec-num- `ext_trie_blake2_256_verify_proof` {#id-ext_trie_blake2_256_verify_proof}

Verifies a key/value pair against a Blake2 256-bit merkle root.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-68}

    (func $ext_trie_blake2_256_verify_proof_version_1
        (param $root i32) (param $proof i64)
        (param $key i64) (param $value i64)
        (result i32))

**Arguments**

- `root`: a pointer to the 256-bit merkle root.

- `proof`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an array containing the node proofs.

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the value.

- `result`: a value equal to _1_ if the proof could be successfully verified or a value equal to _0_ if otherwise.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-13}

    (func $ext_trie_blake2_256_verify_proof_version_2
        (param $root i32) (param $proof i64)
        (param $key i64) (param $value i64)
        (param $version i32) (result i32))

**Arguments**

- `root`: a pointer to the 256-bit merkle root.

- `proof`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an array containing the node proofs.

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the value.

- `version`: the state version ([Definition -def-num-ref-](chap-host-api#defn-state-version)).

- `result`: a value equal to _1_ if the proof could be successfully verified or a value equal to _0_ if otherwise.

### -sec-num- `ext_trie_keccak_256_verify_proof` {#id-ext_trie_keccak_256_verify_proof}

Verifies a key/value pair against a Keccak 256-bit merkle root.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-69}

    (func $ext_trie_keccak_256_verify_proof_version_1
        (param $root i32) (param $proof i64)
        (param $key i64) (param $value i64)
        (result i32))

**Arguments**

- `root`: a pointer to the 256-bit merkle root.

- `proof`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an array containing the node proofs.

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the value.

- `result`: a value equal to _1_ if the proof could be successfully verified or a value equal to _0_ if otherwise.

#### -sec-num- Version 2 - Prototype {#id-version-2-prototype-14}

    (func $ext_trie_keccak_256_verify_proof_version_2
        (param $root i32) (param $proof i64)
        (param $key i64) (param $value i64)
        (param $version i32) (result i32))

**Arguments**

- `root`: a pointer to the 256-bit merkle root.

- `proof`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to an array containing the node proofs.

- `key`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the key.

- `value`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the value.

- `version`: the state version ([Definition -def-num-ref-](chap-host-api#defn-state-version)).

- `result`: a value equal to _1_ if the proof could be successfully verified or a value equal to _0_ if otherwise.

## -sec-num- Miscellaneous {#sect-misc-api}

Interface that provides miscellaneous functions for communicating between the runtime and the node.

### -sec-num- `ext_misc_print_num` {#id-ext_misc_print_num}

Print a number.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-70}

    (func $ext_misc_print_num_version_1 (param $value i64))

**Arguments**

- `value`: the number to be printed.

### -sec-num- `ext_misc_print_utf8` {#id-ext_misc_print_utf8}

Print a valid UTF8 encoded buffer.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-71}

    (func $ext_misc_print_utf8_version_1 (param $data i64))

**Arguments**:

- : a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the valid buffer to be printed.

### -sec-num- `ext_misc_print_hex` {#id-ext_misc_print_hex}

Print any buffer in hexadecimal representation.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-72}

    (func $ext_misc_print_hex_version_1 (param $data i64))

**Arguments**:

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the buffer to be printed.

### -sec-num- `ext_misc_runtime_version` {#id-ext_misc_runtime_version}

Extract the Runtime version of the given Wasm blob by calling `Core_version` ([Section -sec-num-ref-](chap-runtime-api#defn-rt-core-version)). Returns the SCALE encoded runtime version or _None_ ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) if the call fails. This function gets primarily used when upgrading Runtimes.

:::caution
Calling this function is very expensive and should only be done very occasionally. For getting the runtime version, it requires instantiating the Wasm blob ([Section -sec-num-ref-](chap-state#sect-loading-runtime-code)) and calling the `Core_version` function ([Section -sec-num-ref-](chap-runtime-api#defn-rt-core-version)) in this blob.
:::

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-73}

    (func $ext_misc_runtime_version_version_1 (param $data i64) (result i64))

**Arguments**

- `data`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the Wasm blob.

- `result`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the SCALE encoded _Option_ value ([Definition -def-num-ref-](id-cryptography-encoding#defn-option-type)) containing the Runtime version of the given Wasm blob which is encoded as a byte array.

## -sec-num- Allocator {#sect-allocator-api}

The Polkadot Runtime does not include a memory allocator and relies on the Host API for all heap allocations. The beginning of this heap is marked by the `__heap_base` symbol exported by the Polkadot Runtime. No memory should be allocated below that address, to avoid clashes with the stack and data section. The same allocator made accessible by this Host API should be used for any other WASM memory allocations and deallocations outside the runtime e.g. when passing the SCALE-encoded parameters to Runtime API calls.

### -sec-num- `ext_allocator_malloc` {#id-ext_allocator_malloc}

Allocates the given number of bytes and returns the pointer to that memory location.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-74}

    (func $ext_allocator_malloc_version_1 (param $size i32) (result i32))

**Arguments**

- `size`: the size of the buffer to be allocated.

- `result`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the allocated buffer.

### -sec-num- `ext_allocator_free` {#id-ext_allocator_free}

Free the given pointer.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-75}

    (func $ext_allocator_free_version_1 (param $ptr i32))

**Arguments**

- `ptr`: a pointer ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer)) to the memory buffer to be freed.

## -sec-num- Logging {#sect-logging-api}

Interface that provides functions for logging from within the runtime.

###### Definition -def-num- Log Level {#defn-logging-log-level}

:::definition

The **Log Level**, ${L}$, is a varying data type ([Definition -def-num-ref-](id-cryptography-encoding#defn-varrying-data-type)) and implies the emergency of the log. Possible log levels and the corresponding identifier is as follows:

$$
{L}={\left\lbrace\begin{matrix}{0}&\text{Error = 1}\\{1}&\text{Warn = 2}\\{2}&\text{Info = 3}\\{3}&\text{Debug = 4}\\{4}&\text{Trace = 5}\end{matrix}\right.}
$$

:::

### -sec-num- `ext_logging_log` {#id-ext_logging_log}

Request to print a log message on the host. Note that this will be only displayed if the host is enabled to display log messages with given level and target.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-76}

    (func $ext_logging_log_version_1
        (param $level i32) (param $target i64) (param $message i64))

**Arguments**

- `level`: the log level ([Definition -def-num-ref-](chap-host-api#defn-logging-log-level)).

- `target`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the string which contains the path, module or location from where the log was executed.

- `message`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the UTF-8 encoded log message.

### -sec-num- `ext_logging_max_level` {#id-ext_logging_max_level}

Returns the max logging level used by the host.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-max_level}

    (func $ext_logging_max_level_version_1
         (result i32))

**Arguments**

- _None_

**Returns**

- `result`: the max log level ([Definition -def-num-ref-](chap-host-api#defn-logging-log-level)) used by the host.

## -sec-num- Abort Handler {#id-abort-handler}

Interface for aborting the execution of the runtime.

### -sec-num- `ext_panic_handler_abort_on_panic` {#id-ext_panic_handler_abort_on_panic}

Aborts the execution of the runtime with a given message. Note that the message will be only displayed if the host is enabled to display those types of messages, which is implementation specific.

#### -sec-num- Version 1 - Prototype {#id-version-1-prototype-77}

    (func $ext_panic_handler_abort_on_panic_version_1
        (param $message i64))

**Arguments**

- `message`: a pointer-size ([Definition -def-num-ref-](chap-host-api#defn-runtime-pointer-size)) to the UTF-8 encoded message.
