The specs are currently written in a typed subset of python.
There's a library `speclib.py` that provides common functionality that can be used in any specification written in hacspec.

```
Types t ::= int | uint32 | uint64 | uint128 | Tuple[t1,...,tn] | array[t] | bytes
```

```
Expressions e ::=
	    | x		               (variables)
	    | n			       (integer constants in hex)
            | 0x...		       (integer constants in decimal)
	    | f(e1,...en)      	       (call builtin or user-defined function)
            | e1 binop e2              (operators on int and uintN, overloaded)
	      	      		       (binop includes arithmetic: +,-,*,//,%
				       	      	       bit manipulations: <<,>>,&,|
						       comparison: ==, !=, <, >, <=, >= )
	    | uintN(e)		       (convert int to uintN)
            | (e1,...,en)	       (make tuple)
	    | array([e1,...,en])       (make array)
	    | array(x for x in e)      (make array from generator)
            | e[e0]                    (array access)
            | e[e0:e1]                 (array slice)
	    | raise Error("...")       (stop execution with error)
```

```
Statements s ::=
 	   | x : t = e			(variable declaration)
           | def f(x1:t1,...,xn:tn) -> t :
	     	 s		        (function declaration)
	   | x = e                      (assignment)
	   | (x1,..,xn) = e             (tuple assignment)
           | x[i] = e             	(array update)
           | x[i:j] = e           	(array slice update)
           | return e             	(return)
           | if e:
	     	s
	     else:
		s		    	(conditional)
           | for i in range(e):
	     	s		  	(for loop)
           | s
	     s		             	(sequential composition)
           | from x import x1,x2,...,xn (import from external module)
```

## Library functions

```
Builtin functions (hacspec library in speclib.py):

uint8, uint32, uint63, uint128:
  to_int(u:uintN) -> int		(convert uintN to int)
  from_bytes_le(b:bytes) -> uintN       (bytes to uintN)
  to_bytes_le(u:uintN) -> bytes		(uintN to bytes)

array[T]:
  copy(e:array[T]) -> array[T]          (copy array)
  create(d:T,len:int) -> array[T]     	(make array with len elements, each equal to d)
  len(a)	      	                (get length of array)
  concat_blocks(array[array[T]]) -> array[T]
					(flatten array of arrays by concatenation)
  split_blocks(a:array[T],blocksize:int) -> array[array[T]]
  					(split array into blocks of size blocksize;
					 last element may have size < blocksize)
  zip(a:array[T],b:array[U]) -> array[Tuple[T,U]]
					(zip two arrays into an array of tuples;
					 if the two arrays have different lengths,
					 truncate to shorter length)
  enumerate(a:array[T]) -> array[Tuple[int,U]]
					(convert each element x at index i into a pair (i,x))
  uint32s_from_bytes_le(b:bytes) -> array[uint32]
  				        (create array[uint32] from array[bytes])
  uint32s_to_bytes_le(us:array[uint32]) -> bytes
  					(create bytes from array[uint32])
```

# Test Vectors

Test vectors are define in JSON following some schema.
Every schema can be checked either with `mypy` or with [`jsonschema`](http://json-schema.org/specification.html).
Note that `jsonschema` is stricter as it checks the JSON key names while `mypy` only checks types.

Test vector schemata MUST NOT use types other than "string" and "int"/"number".
Integers MUST be base 10.

## MAC

### Mypy TypedDict
Usage example:
`mypy specs/test_vectors/poly1305_test_vectors.py`

```
mac_test = TypedDict('mac_test', {
    'input_len': str,
    'input': str,
    'key' :  str,
    'tag' :  str}
)
```

### JSON Schema
Usage example:
`python spec-checker/check_schema.py mac specs/test_vectors/poly1305_test_vectors.json`

```
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "input_len": {
        "type": "string"
      },
      "input": {
        "type": "string"
      },
      "key": {
        "type": "string"
      },
      "tag": {
        "type": "string"
      }
    },
    "required": [
      "input_len",
      "input",
      "key",
      "tag"
    ]
  },
  "maxProperties": 4
}
```

## Symmetric Encryption

### Mypy TypedDict
Usage example:
`mypy specs/test_vectors/chacha20_test_vectors.py`

```
enc_test = TypedDict('enc_test', {
    'input_len': int,
    'input': str,
    'key' :  str,
    'nonce' :  str,
    'counter' : int,
    'output' :  str})
```

### JSON Schema
Usage example:
`python spec-checker/check_schema.py enc specs/test_vectors/chacha20_test_vectors.json`

```
enc_schema = {
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "input_len": {
        "type": "number"
      },
      "input": {
        "type": "string"
      },
      "key": {
        "type": "string"
      },
      "nonce": {
        "type": "string"
      },
      "counter": {
        "type": "number"
      },
      "output": {
        "type": "string"
      }
    },
    "required": [
      "input_len",
      "input",
      "key",
      "nonce",
      "counter",
      "output"
    ]
  },
  "maxProperties": 6
}
```
