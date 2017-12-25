# (uuid)

The `(uuid)` library can generate and analyze UUIDs.

UUID stands for Universally Unique IDentifiers. They are 128-bit
identifiers standardized by RFC 4122 (though their use predates the
RFC and they are sometimes refered to as GUIDs). No central registry
is required to generate a UUID. An example:
`de7f5de9-bb0b-44ac-a018-e4651868d2ed`.

Although a UUID may look completely random at first sight, it does in
fact have a structure. There is a *variant* field that can be
used to differentiate between different formats. For the RFC 4122
variant there is also a version field that tells you how the UUID was
generated.

This library handles UUIDs as bytevectors and supports converting them
to and from strings. Some systems handle UUIDs on a field-by-field
basis (where the fields are separated by the minus signs in the string
representation). No such conversion routines are provided.

## Background

This library was present in Industria 1.5 and removed from 2.0.0. The
`nil-uuid` and `uuid-namespace-*` exports are now procedures.

## API

```Scheme
(import (uuid))
```

### (time-uuid [node-address])

Generates a time-based UUID. These UUIDs use the system clock together
with a node address which is meant to be unique for each computer
system.

There are three options for the optional *node-address* argument. If
*node-address* is a bytevector of length six then it is used for the
*node* field of the UUID. If *node-address* is the symbol *generate*
then a random bytevector is used (the multicast bit is always set). If
*node-address* is `#f` or omitted then this procedure may use one of
the 48-bit IEEE addresses from the system. If it can't do that, then
it generates a random address that persists across calls.

This procedure uses internal state to reduce the likelihood of
duplicated UUIDs that are caused by a non-monotonous system clock.
This is still not enough to guarantee non-duplicate UUIDs if multiple
programs are generating UUIDs on the same system and at the same time.
This procedure has a maximum throughput that depends on the resolution
of the system time.

You probably want to use `random-uuid` instead, unless you are working
with a namespace.

### (md5-uuid namespace name)

Deterministically generate a UUID based on a name in a namespace, based
on the MD5 hash function.

The *name* can be either a bytevector or a string. If it is a string
it is converted to its UTF-8 representation.

The *namespace* is given as a UUID, of which several are pre-defined
below.

### (sha-1-uuid namespace name)

Deterministically generate a UUID based on a name in a namespace,
based on the SHA-1 cryptographic hash function.

The *name* can be either a bytevector or a string. If it is a string
it is converted to its UTF-8 representation.

The *namespace* is given as a UUID, of which several are pre-defined
below.

```Scheme
(import (uuid))
(uuid->string (md5-uuid (uuid-namespace-dns) "www.example.com"))
;; => "5df41881-3aed-3515-88a7-2f4a814cf09e"
```

### (random-uuid)

This generates a pseudo-random UUID, which has a high probability of
being unique. All the bits are random except for the ones that
identify it as a random UUID.

### (uuid->string uuid)

Convert the *uuid* to its string representation. The hexadecimal
digits will always be lower case.

It raises an assertion violation if *uuid* is not a valid UUID (but no
fields in the UUID are checked).

### (string->uuid string)

Converts a string representation of a UUID into a bytevector.
Non-hexadecimal digits are ignored.

It raises an assertion violation if *string* does not contain a
valid UUID (but no fields in the UUID are checked).

### (uuid-info uuid)

X-ray goggles for your UUIDs. Extracts information from *uuid*. It can
be used to analyze the contents of a UUID. It returns an alist with
various pieces of information, such as the variant and the UUID
version.

What information is returned depends on the variant and version of the
UUID. The fields that are present in the current implementation are
guaranteed to be present in future implementations, though the order
may change and new fields or values may be added. These fields are
defined:

 - `variant` - the variant of the UUID: `nil`, `ncs`, `rfc4122`,
   `microsoft-com`, `reserved` and `unknown`;

 - `version` - the version number of an RFC 4122 UUID;

 - `version-name` - the symbolic version number of
   an RFC 4122 UUID. It is one of these symbols: `time`,
   `dce-security`, `md5`, `random`, or `sha-1`;

 - `reserved-field` - always `2` for a correctly generated UUID;

 - `time` - the UUID generation time as a a SRFI-19 time-utc object;

 - `clock-sequence` - a number that changes when the UUID generator
   detects that the system time may be repeating. It can also be
   randomly generated;

 - `node` - the node address (MAC address) of the system that
   generated the UUID;
 
 - `node-random?` - true when the node address was randomly generated.

```
(import (uuid))
(uuid-info (time-uuid))
;; =>
 ((variant . rfc4122)
  (version . 1)
  (reserved-field . 2)
  (version-name . time)
  (time . #[time time-utc 2099510 1375538665])
  (clock-sequence . 12133)
  (node . #vu8(119 28 30 179 114 29))
  (node-random? . #t))
```

### (nil-uuid)

The special *nil* UUID, which is all zeroes.

### (uuid-namespace-dns)
### (uuid-namespace-url)
### (uuid-namespace-oid)
### (uuid-namespace-x.500)

These procedures return the standard UUIDs of namespaces that are used
with the *md5-uuid* and *sha-1-uuid* procedures. They are used when
the name belongs to a DNS hostname, a URL, an ISO Object ID, or an
X.500 Distinguished Name.
