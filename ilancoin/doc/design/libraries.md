# Libraries

| Name                     | Description |
|--------------------------|-------------|
| *libilancoin_cli*         | RPC client functionality used by *ilancoin-cli* executable |
| *libilancoin_common*      | Home for common functionality shared by different executables and libraries. Similar to *libilancoin_util*, but higher-level (see [Dependencies](#dependencies)). |
| *libilancoin_consensus*   | Consensus functionality used by *libilancoin_node* and *libilancoin_wallet*. |
| *libilancoin_crypto*      | Hardware-optimized functions for data encryption, hashing, message authentication, and key derivation. |
| *libilancoin_kernel*      | Consensus engine and support library used for validation by *libilancoin_node*. |
| *libilancoinqt*           | GUI functionality used by *ilancoin-qt* and *ilancoin-gui* executables. |
| *libilancoin_ipc*         | IPC functionality used by *ilancoin-node*, *ilancoin-wallet*, *ilancoin-gui* executables to communicate when [`-DWITH_MULTIPROCESS=ON`](multiprocess.md) is used. |
| *libilancoin_node*        | P2P and RPC server functionality used by *ilancoind* and *ilancoin-qt* executables. |
| *libilancoin_util*        | Home for common functionality shared by different executables and libraries. Similar to *libilancoin_common*, but lower-level (see [Dependencies](#dependencies)). |
| *libilancoin_wallet*      | Wallet functionality used by *ilancoind* and *ilancoin-wallet* executables. |
| *libilancoin_wallet_tool* | Lower-level wallet functionality used by *ilancoin-wallet* executable. |
| *libilancoin_zmq*         | [ZeroMQ](../zmq.md) functionality used by *ilancoind* and *ilancoin-qt* executables. |

## Conventions

- Most libraries are internal libraries and have APIs which are completely unstable! There are few or no restrictions on backwards compatibility or rules about external dependencies. An exception is *libilancoin_kernel*, which, at some future point, will have a documented external interface.

- Generally each library should have a corresponding source directory and namespace. Source code organization is a work in progress, so it is true that some namespaces are applied inconsistently, and if you look at [`add_library(ilancoin_* ...)`](../../src/CMakeLists.txt) lists you can see that many libraries pull in files from outside their source directory. But when working with libraries, it is good to follow a consistent pattern like:

  - *libilancoin_node* code lives in `src/node/` in the `node::` namespace
  - *libilancoin_wallet* code lives in `src/wallet/` in the `wallet::` namespace
  - *libilancoin_ipc* code lives in `src/ipc/` in the `ipc::` namespace
  - *libilancoin_util* code lives in `src/util/` in the `util::` namespace
  - *libilancoin_consensus* code lives in `src/consensus/` in the `Consensus::` namespace

## Dependencies

- Libraries should minimize what other libraries they depend on, and only reference symbols following the arrows shown in the dependency graph below:

<table><tr><td>

```mermaid

%%{ init : { "flowchart" : { "curve" : "basis" }}}%%

graph TD;

ilancoin-cli[ilancoin-cli]-->libilancoin_cli;

ilancoind[ilancoind]-->libilancoin_node;
ilancoind[ilancoind]-->libilancoin_wallet;

ilancoin-qt[ilancoin-qt]-->libilancoin_node;
ilancoin-qt[ilancoin-qt]-->libilancoinqt;
ilancoin-qt[ilancoin-qt]-->libilancoin_wallet;

ilancoin-wallet[ilancoin-wallet]-->libilancoin_wallet;
ilancoin-wallet[ilancoin-wallet]-->libilancoin_wallet_tool;

libilancoin_cli-->libilancoin_util;
libilancoin_cli-->libilancoin_common;

libilancoin_consensus-->libilancoin_crypto;

libilancoin_common-->libilancoin_consensus;
libilancoin_common-->libilancoin_crypto;
libilancoin_common-->libilancoin_util;

libilancoin_kernel-->libilancoin_consensus;
libilancoin_kernel-->libilancoin_crypto;
libilancoin_kernel-->libilancoin_util;

libilancoin_node-->libilancoin_consensus;
libilancoin_node-->libilancoin_crypto;
libilancoin_node-->libilancoin_kernel;
libilancoin_node-->libilancoin_common;
libilancoin_node-->libilancoin_util;

libilancoinqt-->libilancoin_common;
libilancoinqt-->libilancoin_util;

libilancoin_util-->libilancoin_crypto;

libilancoin_wallet-->libilancoin_common;
libilancoin_wallet-->libilancoin_crypto;
libilancoin_wallet-->libilancoin_util;

libilancoin_wallet_tool-->libilancoin_wallet;
libilancoin_wallet_tool-->libilancoin_util;

classDef bold stroke-width:2px, font-weight:bold, font-size: smaller;
class ilancoin-qt,ilancoind,ilancoin-cli,ilancoin-wallet bold
```
</td></tr><tr><td>

**Dependency graph**. Arrows show linker symbol dependencies. *Crypto* lib depends on nothing. *Util* lib is depended on by everything. *Kernel* lib depends only on consensus, crypto, and util.

</td></tr></table>

- The graph shows what _linker symbols_ (functions and variables) from each library other libraries can call and reference directly, but it is not a call graph. For example, there is no arrow connecting *libilancoin_wallet* and *libilancoin_node* libraries, because these libraries are intended to be modular and not depend on each other's internal implementation details. But wallet code is still able to call node code indirectly through the `interfaces::Chain` abstract class in [`interfaces/chain.h`](../../src/interfaces/chain.h) and node code calls wallet code through the `interfaces::ChainClient` and `interfaces::Chain::Notifications` abstract classes in the same file. In general, defining abstract classes in [`src/interfaces/`](../../src/interfaces/) can be a convenient way of avoiding unwanted direct dependencies or circular dependencies between libraries.

- *libilancoin_crypto* should be a standalone dependency that any library can depend on, and it should not depend on any other libraries itself.

- *libilancoin_consensus* should only depend on *libilancoin_crypto*, and all other libraries besides *libilancoin_crypto* should be allowed to depend on it.

- *libilancoin_util* should be a standalone dependency that any library can depend on, and it should not depend on other libraries except *libilancoin_crypto*. It provides basic utilities that fill in gaps in the C++ standard library and provide lightweight abstractions over platform-specific features. Since the util library is distributed with the kernel and is usable by kernel applications, it shouldn't contain functions that external code shouldn't call, like higher level code targeted at the node or wallet. (*libilancoin_common* is a better place for higher level code, or code that is meant to be used by internal applications only.)

- *libilancoin_common* is a home for miscellaneous shared code used by different Ilancoin Core applications. It should not depend on anything other than *libilancoin_util*, *libilancoin_consensus*, and *libilancoin_crypto*.

- *libilancoin_kernel* should only depend on *libilancoin_util*, *libilancoin_consensus*, and *libilancoin_crypto*.

- The only thing that should depend on *libilancoin_kernel* internally should be *libilancoin_node*. GUI and wallet libraries *libilancoinqt* and *libilancoin_wallet* in particular should not depend on *libilancoin_kernel* and the unneeded functionality it would pull in, like block validation. To the extent that GUI and wallet code need scripting and signing functionality, they should be get able it from *libilancoin_consensus*, *libilancoin_common*, *libilancoin_crypto*, and *libilancoin_util*, instead of *libilancoin_kernel*.

- GUI, node, and wallet code internal implementations should all be independent of each other, and the *libilancoinqt*, *libilancoin_node*, *libilancoin_wallet* libraries should never reference each other's symbols. They should only call each other through [`src/interfaces/`](../../src/interfaces/) abstract interfaces.

## Work in progress

- Validation code is moving from *libilancoin_node* to *libilancoin_kernel* as part of [The libilancoinkernel Project #27587](https://github.com/ilancoin/ilancoin/issues/27587)
