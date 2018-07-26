Shared Libraries
================

## catcoinconsensus

The purpose of this library is to make the verification functionality that is critical to Catcoin's consensus available to other applications, e.g. to language bindings.

### API

The interface is defined in the C header `catcoinconsensus.h` located in  `src/script/catcoinconsensus.h`.

#### Version

`catcoinconsensus_version` returns an `unsigned int` with the API version *(currently at an experimental `0`)*.

#### Script Validation

`catcoinconsensus_verify_script` returns an `int` with the status of the verification. It will be `1` if the input script correctly spends the previous output `scriptPubKey`.

##### Parameters
- `const unsigned char *scriptPubKey` - The previous output script that encumbers spending.
- `unsigned int scriptPubKeyLen` - The number of bytes for the `scriptPubKey`.
- `const unsigned char *txTo` - The transaction with the input that is spending the previous output.
- `unsigned int txToLen` - The number of bytes for the `txTo`.
- `unsigned int nIn` - The index of the input in `txTo` that spends the `scriptPubKey`.
- `unsigned int flags` - The script validation flags *(see below)*.
- `catcoinconsensus_error* err` - Will have the error/success code for the operation *(see below)*.

##### Script Flags
- `catcoinconsensus_SCRIPT_FLAGS_VERIFY_NONE`
- `catcoinconsensus_SCRIPT_FLAGS_VERIFY_P2SH` - Evaluate P2SH ([BIP16](https://github.com/catcoin/bips/blob/master/bip-0016.mediawiki)) subscripts
- `catcoinconsensus_SCRIPT_FLAGS_VERIFY_DERSIG` - Enforce strict DER ([BIP66](https://github.com/catcoin/bips/blob/master/bip-0066.mediawiki)) compliance
- `catcoinconsensus_SCRIPT_FLAGS_VERIFY_NULLDUMMY` - Enforce NULLDUMMY ([BIP147](https://github.com/catcoin/bips/blob/master/bip-0147.mediawiki))
- `catcoinconsensus_SCRIPT_FLAGS_VERIFY_CHECKLOCKTIMEVERIFY` - Enable CHECKLOCKTIMEVERIFY ([BIP65](https://github.com/catcoin/bips/blob/master/bip-0065.mediawiki))
- `catcoinconsensus_SCRIPT_FLAGS_VERIFY_CHECKSEQUENCEVERIFY` - Enable CHECKSEQUENCEVERIFY ([BIP112](https://github.com/catcoin/bips/blob/master/bip-0112.mediawiki))
- `catcoinconsensus_SCRIPT_FLAGS_VERIFY_WITNESS` - Enable WITNESS ([BIP141](https://github.com/catcoin/bips/blob/master/bip-0141.mediawiki))

##### Errors
- `catcoinconsensus_ERR_OK` - No errors with input parameters *(see the return value of `catcoinconsensus_verify_script` for the verification status)*
- `catcoinconsensus_ERR_TX_INDEX` - An invalid index for `txTo`
- `catcoinconsensus_ERR_TX_SIZE_MISMATCH` - `txToLen` did not match with the size of `txTo`
- `catcoinconsensus_ERR_DESERIALIZE` - An error deserializing `txTo`
- `catcoinconsensus_ERR_AMOUNT_REQUIRED` - Input amount is required if WITNESS is used

### Example Implementations
- [NCatcoin](https://github.com/NicolasDorier/NCatcoin/blob/master/NCatcoin/Script.cs#L814) (.NET Bindings)
- [node-libcatcoinconsensus](https://github.com/bitpay/node-libcatcoinconsensus) (Node.js Bindings)
- [java-libcatcoinconsensus](https://github.com/dexX7/java-libcatcoinconsensus) (Java Bindings)
- [catcoinconsensus-php](https://github.com/Bit-Wasp/catcoinconsensus-php) (PHP Bindings)
