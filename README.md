# Abiwan

<a href="https://badge.fury.io/js/abi-wan-kanabi"><img src="https://badge.fury.io/js/abi-wan-kanabi.svg" alt="npm version" height="18"></a>

<details>
<summary>Table of Contents</summary>

- [About](#about)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Usage standalone](#usage-standalone)
  - [Usage with starknet.js](#usage-with-starknetjs)
  - [Demo](#demo)
- [Supported Cairo Types](#supported-cairo-types)
- [Contributing](#contributing)
- [Acknowledgements](#acknowledgements)

</details>

## About

Abiwan is an UNLICENSE standalone TypeScript parser for Cairo smart contracts.
It enables on the fly typechecking and autocompletion for contract calls directly in TypeScript.
Developers can now catch typing mistakes early, prior to executing the call on-chain, and thus enhancing the overall Dapp development experience.

### Cairo versions

Abiwan will support multiple Cairo compiler versions, but not in parallel - different package versions will support different Cairo versions.

| Abiwan                                                        | Cairo compiler                                                                                                                                               |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| [1.0.3](https://www.npmjs.com/package/abi-wan-kanabi/v/1.0.3) | [Cairo v1.0.0](https://github.com/starkware-libs/cairo/releases/tag/v1.0.0) <br> [Cairo v1.1.0](https://github.com/starkware-libs/cairo/releases/tag/v1.1.0) |
| [2.1.0](https://www.npmjs.com/package/abi-wan-kanabi/v/2.1.0) | [Cairo v2.3.0](https://github.com/starkware-libs/cairo/releases/tag/v2.3.0)                                                                                  |

## Getting Started

### Prerequisites

Abiwan dependence only on typescript version 4.9.5 or higher.
Also, it makes use of BigInt, so the `tsconfig.json` should target at least `ES2020`:

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "ESNext"]
  }
}
```

### Usage standalone

To use Abiwan, you must first export your ABI as const in a typescript file

```typescript
export const ABI = [
  //Your ABI here
] as const;
```

If you have a json file containing your contract class, you can use the CLI to generate the typescript file for you:

```bash
npx abi-wan-kanabi --input /path/to/contract_class.json --output /path/to/abi.ts
```

You can then import it in any script and you are set to go:

```typescript
import ABI from "./path/to/abi";
import { call } from "abi-wan-kanabi";
// You'll notice the editor is able to infer the types of the contract's functions
// It'll give you autocompletion and typechecking
const balance = call(ABI, "your_function_name", ["your", "function", "args"]);
```

> If you think that we should be able to import the ABI directly from the json files, we think so too!
> See this typescript [issue](https://github.com/microsoft/TypeScript/issues/32063) and thumb it up!

### Usage with `starknet.js`

Let's say you want to interact with the [Ekubo: Core](https://starkscan.co/contract/0x00000005dd3d2f4429af886cd1a3b08289dbcea99a294197e9eb43b0e0325b4b) contract using starknet.js

You need to first get the **ABI** of the contract and export it in a typescript file, you can do so using one command combining both [`starkli`](https://github.com/xJonathanLEI/starkli) (tested with version 0.2.3) and `npx abi-wan-kanabi`, the command will also print a helpful snippet that you can use to get started

```bash
starkli class-at "0x00000005dd3d2f4429af886cd1a3b08289dbcea99a294197e9eb43b0e0325b4b" --network mainnet | npx abi-wan-kanabi --input /dev/stdin --output abi.ts
```

```javascript
import { Contract, RpcProvider, constants } from "starknet";
import { ABI } from "./abi";

async function main() {
  const address =
    "0x00000005dd3d2f4429af886cd1a3b08289dbcea99a294197e9eb43b0e0325b4b";
  const provider = new RpcProvider({ nodeUrl: constants.NetworkName.SN_MAIN });
  const contract = new Contract(ABI, address, provider).typedv2(ABI);

  const version = await contract.getVersion();
  console.log("version", version);

  // Abiwan is now successfully installed, just start writing your contract
  // function calls (`const ret  = contract.your_function()`) and you'll get
  // helpful editor autocompletion, linting errors ...
  const primary_inteface_id = contract.get_primary_interface_id();
  const protocol_fees_collected = contract.get_protocol_fees_collected("0x1");
}
main().catch(console.error);
```

### Demo

<https://drive.google.com/file/d/1OpIgKlk-okvwJn-dkR2Pq2FvOVwlXTUJ/view?usp=sharing>

##  Supported Cairo Types

Abiwan supports all of Cairo types, here's the mapping between Cairo types and Typescript types

### Primitive Types

| Cairo              | TypeScript                   |
| ------------------ | ---------------------------- |
| `felt252`          | `string \| number \| bigint` |
| `u8 - u32`         | `number \| bigint`           |
| `u64 - u256`       | `number \| bigint \| U256`   |
| `ContractAddress`  | `string`                     |
| `EthAddress`       | `string`                     |
| `ClassHash`        | `string`                     |
| `bool`             | `boolean`                    |
| `()`               | `void`                       |

###  Complex Types

| Cairo                     | TypeScript                                          |
| ------------------------- | --------------------------------------------------- |
| `Option<T>`               | `T \| undefined`                                    |
| `Array<T>`                | `T[]`                                               |
| `Span<T>`                 | `T[]`                                               |
| `tuple (T1, T2, ..., Tn)` | `[T1, T2, ..., Tn]`                                 |
| `struct`                  | an object where keys are struct member names        |
| `enum`                    | a union of objects, each enum variant is an object  |

#### Struct example

**Cairo:**

```cairo
struct TestStruct {
  int128: u128,
  felt: felt252,
  tuple: (u32, u32)
}
```

**Typescript:**

```typescript
{
  int128: number | bigint | Uint256;
  felt: string | number | bigint;
  tuple: [number | bigint, number | bigint];
}
```

#### Enum example

**Cairo:**

```cairo
enum TestEnum {
  int128: u128,
  felt: felt252,
  tuple: (u32, u32),
}
```

**Typescript:**

```typescript
{ int128: number | bigint | Uint256 } |
{ felt: string | number | bigint } |
{ tuple: [number | bigint, number | bigint]}
```

## Contributing

### Run tests

```bash
npm run typecheck
```

### Generate `test/example.ts`

```bash
# First build the example project with `scarb`
cd test/example
scarb build
# Then generate test/example.ts
cd ../..
npm run generate -- --input test/example/target/dev/example_example_contract.contract_class.json --output test/example.ts
```

Contributions on Abiwan are most welcome!
If you are willing to contribute, please get in touch with one of the project lead or via the repositories [Discussions](https://github.com/keep-starknet-strange/abi-wan-kanabi/discussions/categories/general)

## Acknowledgements

### Authors and Contributors

For a full list of all authors and contributors, see [the contributors page](https://github.com/keep-starknet-strange/abi-wan-kanabi/contributors).

### Special mentions

Big thanks and shoutout to [Francesco](https://github.com/fracek)! :clap: who is at the origin of the project!

Also thanks to the awesome Haroune ([@haroune-mohammedi](https://github.com/haroune-mohammedi)) and Thomas ([@thomas-quadratic](https://github.com/thomas-quadratic)) from [Quadratic](https://en.quadratic-labs.com/)!

### Other projects

Abiwan is greatly influenced by the similar project for EVM-compatible contracts [wagmi/abitype](https://github.com/wagmi-dev/abitype).
