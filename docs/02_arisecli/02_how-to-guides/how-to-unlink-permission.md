## Goal

Unlink a linked permission level

## Before you begin

* Install the currently supported version of `arisecli`

* Understand the following:
  * What is an account
  * What is permission level
  * What is an action

## Steps

Remove a linked permission level from an action `transfer` of contract `hodlcontract`

```sh
arisecli set action permission alice hodlcontract transfer NULL
```