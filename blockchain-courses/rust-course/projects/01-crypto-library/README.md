# Project 1: Cryptographic Primitives Library 🔐

**Difficulty:** Beginner | **Estimated Time:** 10-14 hours | **Technologies:** Rust, Cryptography, Testing

## 🎯 Project Overview

Build a comprehensive cryptographic primitives library from scratch using Rust. This project introduces you to the fundamental cryptographic building blocks that power all blockchain systems while mastering Rust's memory safety and performance characteristics.

## 🚀 What You'll Build

A complete cryptographic library with the following components:
- ✅ Hash functions (SHA-256, BLAKE3, Keccak-256)
- ✅ Digital signatures (ECDSA, EdDSA)
- ✅ Merkle trees for data integrity
- ✅ Key derivation functions (PBKDF2, Argon2)
- ✅ Random number generation
- ✅ Base58 and Base64 encoding
- ✅ Comprehensive test suite with benchmarks
- ✅ CLI tool for cryptographic operations

## 🛠️ Technical Requirements

### Core Cryptographic Functions
```rust
// Hash function trait
pub trait Hash {
    type Output;
    fn hash(&self, data: &[u8]) -> Self::Output;
    fn hash_with_salt(&self, data: &[u8], salt: &[u8]) -> Self::Output;
}

// Digital signature trait
pub trait DigitalSignature {
    type PrivateKey;
    type PublicKey;
    type Signature;
    type Error;
    
    fn generate_keypair() -> (Self::PrivateKey, Self::PublicKey);
    fn sign(private_key: &Self::PrivateKey, message: &[u8]) -> Result<Self::Signature, Self::Error>;
    fn verify(public_key: &Self::PublicKey, message: &[u8], signature: &Self::Signature) -> bool;
}
```

### Library Architecture
```rust
// Main library structure
pub mod hash;
pub mod signatures;
pub mod merkle;
pub mod encoding;
pub mod random;
pub mod kdf;
pub mod utils;

// Error handling
#[derive(Debug, thiserror::Error)]
pub enum CryptoError {
    #[error("Invalid key format")]
    InvalidKey,
    #[error("Signature verification failed")]
    InvalidSignature,
    #[error("Hash computation failed")]
    HashError,
    #[error("Encoding error: {0}")]
    EncodingError(String),
}
```

## 📋 Step-by-Step Implementation Guide

### Phase 1: Project Setup and Hash Functions (3-4 hours)

#### 1. Initialize Rust Project
```bash
cargo new crypto-library --lib
cd crypto-library

# Add dependencies to Cargo.toml
```

#### 2. Cargo.toml Configuration
```toml
[package]
name = "crypto-library"
version = "0.1.0"
edition = "2021"

[dependencies]
sha2 = "0.10"
blake3 = "1.4"
tiny-keccak = { version = "2.0", features = ["keccak"] }
ring = "0.16"
ed25519-dalek = "1.0"
k256 = { version = "0.13", features = ["ecdsa"] }
rand = "0.8"
argon2 = "0.5"
base64 = "0.21"
bs58 = "0.4"
thiserror = "1.0"
hex = "0.4"

[dev-dependencies]
criterion = "0.5"
proptest = "1.0"

[[bench]]
name = "crypto_benchmarks"
harness = false

[features]
default = ["std"]
std = []
no-std = []
```

#### 3. Hash Functions Implementation
```rust
// src/hash.rs
use sha2::{Sha256, Digest};
use blake3::Hasher as Blake3Hasher;
use tiny_keccak::{Hasher, Keccak};

/// Hash function trait for unified interface
pub trait Hash {
    type Output: AsRef<[u8]> + Clone;
    
    fn hash(&self, data: &[u8]) -> Self::Output;
    fn hash_with_salt(&self, data: &[u8], salt: &[u8]) -> Self::Output;
}

/// SHA-256 hash function
#[derive(Default)]
pub struct Sha256Hash;

impl Hash for Sha256Hash {
    type Output = [u8; 32];
    
    fn hash(&self, data: &[u8]) -> Self::Output {
        let mut hasher = Sha256::new();
        hasher.update(data);
        hasher.finalize().into()
    }
    
    fn hash_with_salt(&self, data: &[u8], salt: &[u8]) -> Self::Output {
        let mut hasher = Sha256::new();
        hasher.update(salt);
        hasher.update(data);
        hasher.finalize().into()
    }
}

/// BLAKE3 hash function
#[derive(Default)]
pub struct Blake3Hash;

impl Hash for Blake3Hash {
    type Output = [u8; 32];
    
    fn hash(&self, data: &[u8]) -> Self::Output {
        blake3::hash(data).into()
    }
    
    fn hash_with_salt(&self, data: &[u8], salt: &[u8]) -> Self::Output {
        let mut hasher = Blake3Hasher::new();
        hasher.update(salt);
        hasher.update(data);
        hasher.finalize().into()
    }
}

/// Keccak-256 hash function (used by Ethereum)
#[derive(Default)]
pub struct Keccak256Hash;

impl Hash for Keccak256Hash {
    type Output = [u8; 32];
    
    fn hash(&self, data: &[u8]) -> Self::Output {
        let mut hasher = Keccak::v256();
        let mut output = [0u8; 32];
        hasher.update(data);
        hasher.finalize(&mut output);
        output
    }
    
    fn hash_with_salt(&self, data: &[u8], salt: &[u8]) -> Self::Output {
        let mut hasher = Keccak::v256();
        let mut output = [0u8; 32];
        hasher.update(salt);
        hasher.update(data);
        hasher.finalize(&mut output);
        output
    }
}

/// Convenience functions for common hash operations
pub fn sha256(data: &[u8]) -> [u8; 32] {
    Sha256Hash.hash(data)
}

pub fn blake3(data: &[u8]) -> [u8; 32] {
    Blake3Hash.hash(data)
}

pub fn keccak256(data: &[u8]) -> [u8; 32] {
    Keccak256Hash.hash(data)
}

/// Double SHA-256 (used by Bitcoin)
pub fn double_sha256(data: &[u8]) -> [u8; 32] {
    let first_hash = sha256(data);
    sha256(&first_hash)
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_sha256_known_vector() {
        let input = b"hello world";
        let expected = hex::decode("b94d27b9934d3e08a52e52d7da7dabfac484efe37a5380ee9088f7ace2efcde9").unwrap();
        let result = sha256(input);
        assert_eq!(result.as_slice(), expected.as_slice());
    }
    
    #[test]
    fn test_hash_consistency() {
        let data = b"test data";
        let hasher = Sha256Hash;
        
        let hash1 = hasher.hash(data);
        let hash2 = hasher.hash(data);
        assert_eq!(hash1, hash2);
    }
    
    #[test]
    fn test_salt_affects_hash() {
        let data = b"test data";
        let salt1 = b"salt1";
        let salt2 = b"salt2";
        let hasher = Sha256Hash;
        
        let hash1 = hasher.hash_with_salt(data, salt1);
        let hash2 = hasher.hash_with_salt(data, salt2);
        assert_ne!(hash1, hash2);
    }
}
```

### Phase 2: Digital Signatures (3-4 hours)

#### 1. ECDSA Implementation
```rust
// src/signatures/ecdsa.rs
use k256::{
    ecdsa::{SigningKey, VerifyingKey, Signature, signature::Signer, signature::Verifier},
    SecretKey, PublicKey,
};
use rand::rngs::OsRng;
use crate::CryptoError;

/// ECDSA signature implementation using secp256k1 curve
pub struct EcdsaSignature;

impl EcdsaSignature {
    /// Generate a new ECDSA keypair
    pub fn generate_keypair() -> (SigningKey, VerifyingKey) {
        let signing_key = SigningKey::random(&mut OsRng);
        let verifying_key = VerifyingKey::from(&signing_key);
        (signing_key, verifying_key)
    }
    
    /// Sign a message with the private key
    pub fn sign(signing_key: &SigningKey, message: &[u8]) -> Result<Signature, CryptoError> {
        let signature = signing_key.try_sign(message)
            .map_err(|_| CryptoError::InvalidSignature)?;
        Ok(signature)
    }
    
    /// Verify a signature with the public key
    pub fn verify(
        verifying_key: &VerifyingKey,
        message: &[u8],
        signature: &Signature,
    ) -> bool {
        verifying_key.verify(message, signature).is_ok()
    }
    
    /// Convert signing key to bytes
    pub fn signing_key_to_bytes(key: &SigningKey) -> [u8; 32] {
        key.to_bytes().into()
    }
    
    /// Convert verifying key to bytes (compressed)
    pub fn verifying_key_to_bytes(key: &VerifyingKey) -> [u8; 33] {
        key.to_encoded_point(true).as_bytes().try_into().unwrap()
    }
    
    /// Create signing key from bytes
    pub fn signing_key_from_bytes(bytes: &[u8; 32]) -> Result<SigningKey, CryptoError> {
        SigningKey::from_bytes(bytes.into())
            .map_err(|_| CryptoError::InvalidKey)
    }
    
    /// Create verifying key from bytes
    pub fn verifying_key_from_bytes(bytes: &[u8]) -> Result<VerifyingKey, CryptoError> {
        VerifyingKey::from_sec1_bytes(bytes)
            .map_err(|_| CryptoError::InvalidKey)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_ecdsa_sign_verify() {
        let (signing_key, verifying_key) = EcdsaSignature::generate_keypair();
        let message = b"test message";
        
        let signature = EcdsaSignature::sign(&signing_key, message).unwrap();
        let is_valid = EcdsaSignature::verify(&verifying_key, message, &signature);
        
        assert!(is_valid);
    }
    
    #[test]
    fn test_ecdsa_invalid_signature() {
        let (_, verifying_key) = EcdsaSignature::generate_keypair();
        let (other_signing_key, _) = EcdsaSignature::generate_keypair();
        let message = b"test message";
        
        let signature = EcdsaSignature::sign(&other_signing_key, message).unwrap();
        let is_valid = EcdsaSignature::verify(&verifying_key, message, &signature);
        
        assert!(!is_valid);
    }
    
    #[test]
    fn test_key_serialization() {
        let (signing_key, verifying_key) = EcdsaSignature::generate_keypair();
        
        // Test signing key serialization
        let signing_bytes = EcdsaSignature::signing_key_to_bytes(&signing_key);
        let restored_signing = EcdsaSignature::signing_key_from_bytes(&signing_bytes).unwrap();
        
        // Test verifying key serialization
        let verifying_bytes = EcdsaSignature::verifying_key_to_bytes(&verifying_key);
        let restored_verifying = EcdsaSignature::verifying_key_from_bytes(&verifying_bytes).unwrap();
        
        // Test that restored keys work
        let message = b"test message";
        let signature = EcdsaSignature::sign(&restored_signing, message).unwrap();
        let is_valid = EcdsaSignature::verify(&restored_verifying, message, &signature);
        
        assert!(is_valid);
    }
}
```

#### 2. EdDSA Implementation
```rust
// src/signatures/eddsa.rs
use ed25519_dalek::{Keypair, PublicKey, SecretKey, Signature, Signer, Verifier};
use rand::rngs::OsRng;
use crate::CryptoError;

/// EdDSA signature implementation using Ed25519 curve
pub struct EddsaSignature;

impl EddsaSignature {
    /// Generate a new EdDSA keypair
    pub fn generate_keypair() -> Keypair {
        Keypair::generate(&mut OsRng)
    }
    
    /// Sign a message with the private key
    pub fn sign(keypair: &Keypair, message: &[u8]) -> Signature {
        keypair.sign(message)
    }
    
    /// Verify a signature with the public key
    pub fn verify(public_key: &PublicKey, message: &[u8], signature: &Signature) -> bool {
        public_key.verify(message, signature).is_ok()
    }
    
    /// Convert keypair to bytes
    pub fn keypair_to_bytes(keypair: &Keypair) -> [u8; 64] {
        keypair.to_bytes()
    }
    
    /// Convert public key to bytes
    pub fn public_key_to_bytes(public_key: &PublicKey) -> [u8; 32] {
        public_key.to_bytes()
    }
    
    /// Create keypair from bytes
    pub fn keypair_from_bytes(bytes: &[u8; 64]) -> Result<Keypair, CryptoError> {
        Keypair::from_bytes(bytes).map_err(|_| CryptoError::InvalidKey)
    }
    
    /// Create public key from bytes
    pub fn public_key_from_bytes(bytes: &[u8; 32]) -> Result<PublicKey, CryptoError> {
        PublicKey::from_bytes(bytes).map_err(|_| CryptoError::InvalidKey)
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_eddsa_sign_verify() {
        let keypair = EddsaSignature::generate_keypair();
        let message = b"test message";
        
        let signature = EddsaSignature::sign(&keypair, message);
        let is_valid = EddsaSignature::verify(&keypair.public, message, &signature);
        
        assert!(is_valid);
    }
    
    #[test]
    fn test_eddsa_invalid_signature() {
        let keypair1 = EddsaSignature::generate_keypair();
        let keypair2 = EddsaSignature::generate_keypair();
        let message = b"test message";
        
        let signature = EddsaSignature::sign(&keypair1, message);
        let is_valid = EddsaSignature::verify(&keypair2.public, message, &signature);
        
        assert!(!is_valid);
    }
}
```

### Phase 3: Merkle Trees (2-3 hours)

#### 1. Merkle Tree Implementation
```rust
// src/merkle.rs
use crate::hash::{Hash, Sha256Hash};
use std::collections::VecDeque;

/// Merkle tree implementation for data integrity verification
#[derive(Debug, Clone)]
pub struct MerkleTree<H: Hash> {
    hasher: H,
    leaves: Vec<H::Output>,
    tree: Vec<Vec<H::Output>>,
    root: Option<H::Output>,
}

/// Merkle proof for verifying inclusion of data in the tree
#[derive(Debug, Clone)]
pub struct MerkleProof<T> {
    pub leaf_hash: T,
    pub proof_hashes: Vec<T>,
    pub proof_directions: Vec<bool>, // true = right, false = left
    pub root_hash: T,
}

impl<H: Hash> MerkleTree<H> {
    /// Create a new Merkle tree with the given hasher
    pub fn new(hasher: H) -> Self {
        Self {
            hasher,
            leaves: Vec::new(),
            tree: Vec::new(),
            root: None,
        }
    }
    
    /// Build the Merkle tree from a list of data items
    pub fn build(&mut self, data: &[&[u8]]) -> Result<(), crate::CryptoError> {
        if data.is_empty() {
            return Err(crate::CryptoError::HashError);
        }
        
        // Hash all leaf data
        self.leaves = data.iter().map(|item| self.hasher.hash(item)).collect();
        
        // Build the tree bottom-up
        self.tree.clear();
        self.tree.push(self.leaves.clone());
        
        let mut current_level = self.leaves.clone();
        
        while current_level.len() > 1 {
            let mut next_level = Vec::new();
            
            for chunk in current_level.chunks(2) {
                let combined_hash = if chunk.len() == 2 {
                    self.combine_hashes(&chunk[0], &chunk[1])
                } else {
                    // Odd number of nodes, duplicate the last one
                    self.combine_hashes(&chunk[0], &chunk[0])
                };
                next_level.push(combined_hash);
            }
            
            self.tree.push(next_level.clone());
            current_level = next_level;
        }
        
        self.root = current_level.into_iter().next();
        Ok(())
    }
    
    /// Get the root hash of the tree
    pub fn root(&self) -> Option<&H::Output> {
        self.root.as_ref()
    }
    
    /// Generate a proof for the inclusion of data at the given index
    pub fn generate_proof(&self, index: usize) -> Result<MerkleProof<H::Output>, crate::CryptoError> {
        if index >= self.leaves.len() {
            return Err(crate::CryptoError::HashError);
        }
        
        let mut proof_hashes = Vec::new();
        let mut proof_directions = Vec::new();
        let mut current_index = index;
        
        // Traverse from leaf to root, collecting sibling hashes
        for level in 0..self.tree.len() - 1 {
            let sibling_index = if current_index % 2 == 0 {
                current_index + 1
            } else {
                current_index - 1
            };
            
            if sibling_index < self.tree[level].len() {
                proof_hashes.push(self.tree[level][sibling_index].clone());
                proof_directions.push(current_index % 2 == 0); // true if we're left child
            }
            
            current_index /= 2;
        }
        
        Ok(MerkleProof {
            leaf_hash: self.leaves[index].clone(),
            proof_hashes,
            proof_directions,
            root_hash: self.root.as_ref().unwrap().clone(),
        })
    }
    
    /// Verify a Merkle proof
    pub fn verify_proof(&self, proof: &MerkleProof<H::Output>) -> bool {
        let mut current_hash = proof.leaf_hash.clone();
        
        for (sibling_hash, is_left) in proof.proof_hashes.iter().zip(&proof.proof_directions) {
            current_hash = if *is_left {
                self.combine_hashes(&current_hash, sibling_hash)
            } else {
                self.combine_hashes(sibling_hash, &current_hash)
            };
        }
        
        current_hash.as_ref() == proof.root_hash.as_ref()
    }
    
    /// Combine two hashes to create a parent hash
    fn combine_hashes(&self, left: &H::Output, right: &H::Output) -> H::Output {
        let mut combined = Vec::new();
        combined.extend_from_slice(left.as_ref());
        combined.extend_from_slice(right.as_ref());
        self.hasher.hash(&combined)
    }
}

/// Convenience function to create a SHA-256 based Merkle tree
pub fn create_sha256_merkle_tree(data: &[&[u8]]) -> Result<MerkleTree<Sha256Hash>, crate::CryptoError> {
    let mut tree = MerkleTree::new(Sha256Hash);
    tree.build(data)?;
    Ok(tree)
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_merkle_tree_creation() {
        let data = vec![b"data1".as_slice(), b"data2", b"data3", b"data4"];
        let tree = create_sha256_merkle_tree(&data).unwrap();
        
        assert!(tree.root().is_some());
    }
    
    #[test]
    fn test_merkle_proof_verification() {
        let data = vec![b"data1".as_slice(), b"data2", b"data3", b"data4"];
        let tree = create_sha256_merkle_tree(&data).unwrap();
        
        // Generate and verify proof for each leaf
        for i in 0..data.len() {
            let proof = tree.generate_proof(i).unwrap();
            assert!(tree.verify_proof(&proof));
        }
    }
    
    #[test]
    fn test_merkle_proof_invalid() {
        let data1 = vec![b"data1".as_slice(), b"data2", b"data3", b"data4"];
        let data2 = vec![b"other1".as_slice(), b"other2", b"other3", b"other4"];
        
        let tree1 = create_sha256_merkle_tree(&data1).unwrap();
        let tree2 = create_sha256_merkle_tree(&data2).unwrap();
        
        // Proof from tree1 should not verify against tree2
        let proof = tree1.generate_proof(0).unwrap();
        assert!(!tree2.verify_proof(&proof));
    }
}
```

### Phase 4: CLI Tool and Benchmarks (2-3 hours)

#### 1. CLI Application
```rust
// src/bin/crypto-cli.rs
use clap::{App, Arg, SubCommand};
use crypto_library::{hash, signatures, encoding, CryptoError};
use std::fs;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    let matches = App::new("Crypto CLI")
        .version("1.0")
        .about("Cryptographic operations CLI tool")
        .subcommand(
            SubCommand::with_name("hash")
                .about("Hash operations")
                .arg(Arg::with_name("algorithm")
                    .short("a")
                    .long("algorithm")
                    .value_name("ALGO")
                    .help("Hash algorithm (sha256, blake3, keccak256)")
                    .takes_value(true)
                    .default_value("sha256"))
                .arg(Arg::with_name("input")
                    .short("i")
                    .long("input")
                    .value_name("FILE")
                    .help("Input file")
                    .takes_value(true))
                .arg(Arg::with_name("text")
                    .short("t")
                    .long("text")
                    .value_name("TEXT")
                    .help("Input text")
                    .takes_value(true))
        )
        .subcommand(
            SubCommand::with_name("sign")
                .about("Digital signature operations")
                .arg(Arg::with_name("algorithm")
                    .short("a")
                    .long("algorithm")
                    .value_name("ALGO")
                    .help("Signature algorithm (ecdsa, eddsa)")
                    .takes_value(true)
                    .default_value("ecdsa"))
                .arg(Arg::with_name("generate")
                    .short("g")
                    .long("generate")
                    .help("Generate new keypair"))
                .arg(Arg::with_name("sign")
                    .short("s")
                    .long("sign")
                    .value_name("MESSAGE")
                    .help("Sign message")
                    .takes_value(true))
        )
        .get_matches();
    
    match matches.subcommand() {
        ("hash", Some(hash_matches)) => handle_hash_command(hash_matches)?,
        ("sign", Some(sign_matches)) => handle_sign_command(sign_matches)?,
        _ => {
            println!("Use --help for usage information");
        }
    }
    
    Ok(())
}

fn handle_hash_command(matches: &clap::ArgMatches) -> Result<(), Box<dyn std::error::Error>> {
    let algorithm = matches.value_of("algorithm").unwrap();
    
    let data = if let Some(file_path) = matches.value_of("input") {
        fs::read(file_path)?
    } else if let Some(text) = matches.value_of("text") {
        text.as_bytes().to_vec()
    } else {
        return Err("Either --input or --text must be provided".into());
    };
    
    let hash_result = match algorithm {
        "sha256" => hex::encode(hash::sha256(&data)),
        "blake3" => hex::encode(hash::blake3(&data)),
        "keccak256" => hex::encode(hash::keccak256(&data)),
        _ => return Err("Unsupported hash algorithm".into()),
    };
    
    println!("{}: {}", algorithm.to_uppercase(), hash_result);
    Ok(())
}

fn handle_sign_command(matches: &clap::ArgMatches) -> Result<(), Box<dyn std::error::Error>> {
    let algorithm = matches.value_of("algorithm").unwrap();
    
    if matches.is_present("generate") {
        match algorithm {
            "ecdsa" => {
                let (signing_key, verifying_key) = signatures::ecdsa::EcdsaSignature::generate_keypair();
                let signing_bytes = signatures::ecdsa::EcdsaSignature::signing_key_to_bytes(&signing_key);
                let verifying_bytes = signatures::ecdsa::EcdsaSignature::verifying_key_to_bytes(&verifying_key);
                
                println!("Private Key: {}", hex::encode(signing_bytes));
                println!("Public Key: {}", hex::encode(verifying_bytes));
            }
            "eddsa" => {
                let keypair = signatures::eddsa::EddsaSignature::generate_keypair();
                let keypair_bytes = signatures::eddsa::EddsaSignature::keypair_to_bytes(&keypair);
                let public_bytes = signatures::eddsa::EddsaSignature::public_key_to_bytes(&keypair.public);
                
                println!("Keypair: {}", hex::encode(keypair_bytes));
                println!("Public Key: {}", hex::encode(public_bytes));
            }
            _ => return Err("Unsupported signature algorithm".into()),
        }
    }
    
    Ok(())
}
```

#### 2. Performance Benchmarks
```rust
// benches/crypto_benchmarks.rs
use criterion::{black_box, criterion_group, criterion_main, Criterion, BenchmarkId};
use crypto_library::hash::{sha256, blake3, keccak256};
use crypto_library::signatures::{ecdsa::EcdsaSignature, eddsa::EddsaSignature};

fn hash_benchmarks(c: &mut Criterion) {
    let data_sizes = vec![64, 256, 1024, 4096, 16384];
    
    for size in data_sizes {
        let data = vec![0u8; size];
        
        c.bench_with_input(
            BenchmarkId::new("sha256", size),
            &data,
            |b, data| b.iter(|| sha256(black_box(data)))
        );
        
        c.bench_with_input(
            BenchmarkId::new("blake3", size),
            &data,
            |b, data| b.iter(|| blake3(black_box(data)))
        );
        
        c.bench_with_input(
            BenchmarkId::new("keccak256", size),
            &data,
            |b, data| b.iter(|| keccak256(black_box(data)))
        );
    }
}

fn signature_benchmarks(c: &mut Criterion) {
    let message = b"benchmark message";
    
    // ECDSA benchmarks
    let (ecdsa_signing, ecdsa_verifying) = EcdsaSignature::generate_keypair();
    let ecdsa_signature = EcdsaSignature::sign(&ecdsa_signing, message).unwrap();
    
    c.bench_function("ecdsa_sign", |b| {
        b.iter(|| EcdsaSignature::sign(black_box(&ecdsa_signing), black_box(message)))
    });
    
    c.bench_function("ecdsa_verify", |b| {
        b.iter(|| EcdsaSignature::verify(
            black_box(&ecdsa_verifying),
            black_box(message),
            black_box(&ecdsa_signature)
        ))
    });
    
    // EdDSA benchmarks
    let eddsa_keypair = EddsaSignature::generate_keypair();
    let eddsa_signature = EddsaSignature::sign(&eddsa_keypair, message);
    
    c.bench_function("eddsa_sign", |b| {
        b.iter(|| EddsaSignature::sign(black_box(&eddsa_keypair), black_box(message)))
    });
    
    c.bench_function("eddsa_verify", |b| {
        b.iter(|| EddsaSignature::verify(
            black_box(&eddsa_keypair.public),
            black_box(message),
            black_box(&eddsa_signature)
        ))
    });
}

criterion_group!(benches, hash_benchmarks, signature_benchmarks);
criterion_main!(benches);
```

## 📚 Learning Outcomes

After completing this project, you will have:
- Implemented cryptographic primitives from scratch in Rust
- Mastered Rust's ownership system and memory safety
- Built secure, performant cryptographic operations
- Created comprehensive test suites and benchmarks
- Understood the building blocks of blockchain security
- Applied error handling and type safety in cryptographic contexts

---

**Ready to build your crypto library?** This project provides the cryptographic foundation for all blockchain development and introduces you to Rust's powerful type system and performance characteristics.

**Next Project:** [Blockchain Node](../02-blockchain-node/README.md) - Build a basic blockchain using your cryptographic primitives.

---

*This project demonstrates essential cryptographic and Rust skills that every blockchain infrastructure developer needs. Focus on understanding both the cryptography and Rust concepts!*
