# threshold_crypto

[![crates.io](https://img.shields.io/crates/v/threshold_crypto.svg)](https://crates.io/crates/threshold_crypto)
[![Documentation](https://docs.rs/threshold_crypto/badge.svg)](https://docs.rs/threshold_crypto)
[![Build Status](https://travis-ci.org/poanetwork/threshold_crypto.svg?branch=master)](https://travis-ci.org/poanetwork/threshold_crypto)

A pairing-based threshold cryptosystem for collaborative decryption and signatures.

This is a fork from `threshold_crypto` which updates the dependencies to the newest version. Apart from that the code tries ot be as close as possible to retain the security as shown by the original security audit.

The `threshold_bls` crate provides cryptographic keys with methods for signing and encrypting messages, as well as key sets for _threshold_ signatures and threshold encryption.

The threshold signature scheme is described in [Threshold Signatures, Multisignatures and Blind Signatures Based on the Gap-Diffie-Hellman-Group Signature Scheme](https://www.iacr.org/archive/pkc2003/25670031/25670031.pdf) by Alexandra Boldyreva. This paper extends [Boneh-Lynn-Shacham](https://www.iacr.org/archive/asiacrypt2001/22480516.pdf) signatures to the threshold setting. Message encryption uses the [scheme by Baek and Zhang](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.119.1717&rep=rep1&type=pdf).
Our implementation is based on the [`pairing`](https://crates.io/crates/pairing) elliptic curve library.

In a network environment, messages are signed and encrypted, and key and signature shares are distributed to network participants. A message can be decrypted and authenticated only with cooperation from at least `threshold +
1` nodes.

## Security Audit

An [official security audit](https://github.com/poanetwork/wiki/wiki/Threshold-Crypto-Audit) has been completed on the original `threshold_crypto` by [Jean-Philippe Aumasson](https://aumasson.jp/). No exploitable security issues were found, and potential improvements have been addressed. Outdated dependencies mentioned in the audit were updated in commit [54026f5](https://github.com/poanetwork/threshold_crypto/commit/54026f5fe7e0b5a52e446ac01a50469da1f15a71).

## Usage

`Cargo.toml`:

```toml
[dependencies]
threshold_crypto = { version = "0.4", git = "https://github.com/poanetwork/threshold_crypto" }
```

`main.rs`:

```rust
extern crate rand;
extern crate threshold_crypto;

use threshold_crypto::SecretKey;

/// Very basic secret key usage.
fn main() {
    let sk0 = SecretKey::random();
    let sk1 = SecretKey::random();

    let pk0 = sk0.public_key();

    let msg0 = b"Real news";
    let msg1 = b"Fake news";

    assert!(pk0.verify(&sk0.sign(msg0), msg0));
    assert!(!pk0.verify(&sk1.sign(msg0), msg0)); // Wrong key.
    assert!(!pk0.verify(&sk0.sign(msg1), msg0)); // Wrong message.
}
```

### Testing

Run tests with:

```
$ cargo test
```

### Examples

Run examples from the [`examples`](examples) directory using:

```
$ cargo run --example <example name>
```

Also see the
[distributed_key_generation](https://github.com/poanetwork/threshold_crypto/blob/d81953b55d181311c2a4eed2b6c34059fcf3fdae/src/poly.rs#L967)
test.

## Application Details

The basic usage outline is:
* choose a threshold value `t`
* create a key set
* distribute `N` secret key shares among the participants
* publish the public master key

A third party can now encrypt a message to the public master key
and any set of `t + 1` participants *(but no fewer!)* can collaborate to
decrypt it. Also, any set of `t + 1` participants can collaborate to sign a message,
producing a signature that is verifiable with the public master key.

In this system, a signature is unique and independent of
the set of participants that produced it. If `S1` and `S2` are
signatures for the same message, produced by two different sets of `t + 1`
secret key share holders, both signatures will be valid AND
equal. This is useful in some applications, for example a message signature can serve as a pseudorandom number unknown to anyone until `t + 1` participants agree to reveal it.

In its simplest form, threshold_crypto requires a trusted dealer to
produce and distribute the secret key shares. However, keys can be produced so that only the corresponding participant knows their secret in the end.  This crate
includes the basic tools to implement such a *Distributed Key Generation*
scheme.

A major application for this library is within a distributed network that
must tolerate up to `t` adversarial (malicious or faulty) nodes. Because `t +
1` nodes are required to sign or reveal information, messages can be trusted
by third-parties as representing the consensus of the network.

### Documentation

* [crate documentation](https://docs.rs/threshold_crypto/)
* [crates.io package](https://crates.io/crates/threshold_crypto) 

## Performance

Benchmarking functionality is kept in the [`benches` directory](benches). You
can run the benchmarks with the following command:

```
$ RUSTFLAGS="-C target_cpu=native" cargo bench
```

We use the [`criterion`](https://crates.io/crates/criterion) benchmarking library.

### Mock cryptography

To speed up automatic tests of crates depending on `threshold_crypto`, the `use-insecure-test-only-mock-crypto` feature is available. **Activating this feature will effectively disable encryption and should only be used during tests!** Essentially, the underlying elliptic curves will be replaced by small finite fields, yielding a 10-200X speed-up in execution. The resulting ciphers can be trivially broken in a number of ways and should never be used in production.

## License

For now the code is essentially the same as `threshold_crypto`, meaning that the same licenses apply. However, if for any reason the work deviates from the original work to a degree that it constitutes its own work, the authors reserve the right to apply other licenses in the future. As of now, as `threshold_crypto`, the work in this repository is licensed under either of:

* Apache License, Version 2.0, ([LICENSE-APACHE](LICENSE-APACHE) or http://www.apache.org/licenses/LICENSE-2.0)
* MIT license ([LICENSE-MIT](LICENSE-MIT) or http://opensource.org/licenses/MIT)

at your option.

## Contributing

See the [CONTRIBUTING](CONTRIBUTING.md) document for contribution, testing and
pull request protocol.

Unless you explicitly state otherwise, any contribution intentionally
submitted for inclusion in the work by you, as defined in the Apache-2.0
license, shall be dual licensed as above, without any additional terms or
conditions.
