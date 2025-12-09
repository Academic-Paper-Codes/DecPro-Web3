# DecPro

DecPro is a research prototype that implements two privacy-preserving data
processing schemes, DecPro-I and DecPro-II. Both constructions rely on
multi-authority attribute-based encryption and support expressive data access
policies with fine-grained execution control.

## Key Features

- Multi-authority setup with shared attribute universes.
- Registration workflow for issuing user-specific key material.
- Upload, Check, and Process procedures with consistent serialization helpers.
- Reference implementations for both matrix-based (DecPro-I) and tree-based
  (DecPro-II) policy representations.

## Project Structure

```text
DecPro/
├── src/
│   ├── core/
│   │   ├── abe_types.py       # Attribute references and MA-ABE type aliases
│   │   ├── access_matrix.py   # LSSS matrix tools for DecPro-I policies
│   │   ├── access_tree.py     # Access tree helpers for DecPro-II
│   │   ├── bilinear_group.py  # Pairing-friendly group initialisation
│   │   ├── hash_functions.py  # Domain-separated hash wrappers (H1/H2)
│   │   ├── homomorphic_enc.py # Partially homomorphic encryption interface
│   │   ├── ma_abe.py          # Multi-authority ABE driver and key material
│   │   ├── ore.py             # Order-revealing encryption adapter
│   │   ├── prf.py             # Pseudorandom function abstraction
│   │   └── symmetric_enc.py   # Symmetric encryption and AEAD wrappers
│   ├── schemes/
│   │   ├── base.py            # Shared logic for setup/register/upload/check/process
│   │   ├── decpro_i/
│   │   │   ├── scheme.py      # Concrete DecPro-I orchestrator
│   │   │   ├── setup.py       # Authority bootstrap workflow
│   │   │   ├── register.py    # User credential issuance
│   │   │   ├── upload.py      # Ciphertext generation with matrix policies
│   │   │   ├── check.py       # Access decision evaluation
│   │   │   └── process.py     # Record retrieval under granted privileges
│   │   └── decpro_ii/
│   │       ├── scheme.py      # Access-tree-based variant
│   │       ├── setup.py
│   │       ├── register.py
│   │       ├── upload.py
│   │       ├── check.py
│   │       └── process.py
│   ├── tests/
│   │   ├── test_decpro_schemes.py # High-level protocol regression tests
│   │   └── test_ma_abe.py         # Core MA-ABE component tests
│   └── utils/
│       ├── config.py          # Centralised security parameters
│       ├── lagrange.py        # Lagrange interpolation helper routines
│       └── serialization.py   # Binary serialisation for keys and ciphertexts
├── requirements.txt # Python dependencies
└── README.md
```

## Prerequisites

- Python 3.9 or newer
- A virtual environment manager (such as `venv` or `conda`)

## Installation

```bash
python -m venv .venv
source .venv/bin/activate            # On Windows use: .venv\Scripts\activate
pip install --upgrade pip
pip install -r requirements.txt
```

## Running the Test Suite

```bash
pytest src/tests
```

The tests provide end-to-end coverage for the core cryptographic primitives,
policy handling, and both DecPro variants. They also serve as executable usage
examples.

## Quick Start

The snippet below demonstrates a minimal DecPro-I flow. Ensure the repository
root is on `PYTHONPATH` (for example, run the script from the project root).

```python
from src.schemes import AccessLevel, DecProI
from src.core.abe_types import AttributeRef
from src.core.access_matrix import LSSSMatrix

# 1. Setup authorities and publish group parameters
scheme = DecProI()
scheme.setup_authority("AUTH", ["role_admin"])

# 2. Issue user credentials
user_key = scheme.register({"AUTH": ["role_admin"]})

# 3. Upload a dataset under an access matrix
policy = LSSSMatrix([([1], AttributeRef("AUTH", "role_admin"))])
bundle = scheme.upload(policy, ["record-1", "record-2"])

# 4. Evaluate user permissions
decision = scheme.check(policy, [AttributeRef("AUTH", "role_admin")])
assert decision.level == AccessLevel.PLAINTEXT

# 5. Process the ciphertext with the granted privileges
result = scheme.process(bundle.ciphertext, user_key)
print(result.parsed.decode("utf-8"))
```

Switching to DecPro-II only requires importing `DecProII` and constructing an
access tree instead of a matrix. Both schemes expose the same public API for
setup, registration, upload, authorization checks, and ciphertext processing.

