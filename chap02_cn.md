
2. Detailed Design 设计细节

2.1 High-Level Architecture 高级架构

We take Ethereum blockchain as an example to briefly discuss the overall process of an
on-demand data query initiated by a user contract.
It looks similar to the request & response pattern, however, it is an asynchronous process from user contract’s point of view:

我们以Ethereum区块链为例，简要讨论由用户合约发起的按需数据查询的整个过程。
它看起来类似于请求和响应模式，然而，从用户合约的角度来看，它是一个异步的过程:

①: User contract makes a data query request through a message call to
DOS on-chain system (a bunch of smart contracts open sourced and published with well documentation provided to developers), specifically the DOS Proxy Contract;

用户合约通过 对 DOS 链上系统的消息调用发出数据查询请求 (许多智能合约都是开源的并发布的，并向开发人员提供了良好的文档)， 具体是DOS代理合约；


②: DOS Proxy Contract triggers an event along with query parameters;
  DOS代理合约 触发带有查询参数的事件


③: DOS clients (off-chain part of DOS Network running by users), which keep monitoring the blockchain for the defined event, are notified. Ideally there would be thousands of DOS instances running, out of which a registered group will be randomly selected, by means of the distributed randomness engine built with verifiable random function (VRF).

DOS客户端(用户运行的DOS链下部分)， 使用通知的方式保持监控区块链定义的事件。
理想情况下，应该有数千个DOS实例在运行，从这些实例中随机选择一个已注册的组，利用可验证随机函数(VRF)
构建分布式随机引擎。

④ & ⑤: Members in the selected group do the due diligence, calling a web api, performing a computation, or executing a configured script concurrently;

选定组中的成员进行尽职调查， 请求网络api， 执行计算或同时执行已配置的脚本;


⑥: They will reach "in-group" consensus by the t-out-of-n threshold signature algorithm and report back the agreed result to DOS on-chain system, as long as more than t members in the randomly selected group are honest. The selected group members’ identity and QoS (responsiveness/correctness, etc.) performance will be recorded on-chain, for monitoring and data analysis purposes.

他们将达成“in-group”共识， 通过 t-out-n 门限签名算法，并将协议结果反馈给DOS链上系统，
只要随机选取的组中有超过t个成员是诚实的。 选择的组成员的身份和QoS(响应性/正确性等)性能将被记录在链上，
用于监测和数据分析。


⑦: DOS Proxy Contract notifies the user contract that the result is ready, by calling a callback function provided by user contract.

DOS代理合约通过调用用户合约提供的回调函数通知用户合约结果已经就绪。

** The overall workflow of verifiable computation oracle looks similar and also make use
of the distributed randomness engine, but with several differences for step 4~6.
We’ll discuss the details in later sections.

可验证计算oracle的整体工作流程类似，也使用了分布式随机性引擎，但步骤4~6有几个不同之处。
我们将在后面几节中讨论细节。


2.2 On-Chain Detailed Design 链上设计细节

2.2.1 Proxy System 代理系统

The proxy system provides standard on-chain interfaces to user contracts and will
asynchronously callback to user contracts once the response is ready. The interface
provided to user contracts is universal and simple as demonstrated below:

代理系统为用户合约提供标准的链上接口，一旦响应就绪，就将异步回调给用户合约。
提供给用户合约的接口是通用的、简单的，如下所示:

- Making a query request to DOS Proxy Contract;

向DOS代理合约发送查询请求;

- Consuming the result backfilled from `__callback__` function to finish some
post-processing work.

使用从 `__callback__` 函数中回填的结果，来完成一些后续处理工作。

```
code
code
```

Alpha version of DOS on-chain contracts have been open-sourced on Github already
and will be keeping updated.

DOS链上合约的Alpha版本已经在Github上开源并会不断更新。


2.2.2 On-chain Governance Systems 链上治理系统

Monitoring system 监控系统

Monitoring system is proposed to keep on-chain records of the off-chain DOS
instances’ QoS (Quality-of-Service) metrics and network stats, including:

使用监控系统来保持链上的链下DOS实例的QoS(服务质量)指标和网络统计数据的记录，包括：

- Random number generated by latest round’s selected off-chain group, which
could serve as a new kind of on-chain random source;

由最新一轮选择的链下组产生的随机数，可以作为一种新型的链上随机源;

- Group size, number of registered groups, number of times each registered
group has been selected, uptime and decomposition time to get rid of
adaptive adversaries, etc.;

组的大小，注册组的数量，每次注册的小组已被选中的次数，正常运行时间和解散时间，自动排除对手等；

- Payment, weight percentage, and callback delay stats of processed and
unhandled query requests;

支付、权重百分比、回调延迟统计和未处理的查询请求;

- Quality score of registered off-chain DOS instances including correctness and
responsiveness of their reported results - instances with extremely bad quality
score will be excluded from off-chain consensus protocol and payment
process;

已注册的链下DOS实例的质量分数，包括正确率和报告结果的响应率 —— 质量非常差的实例
评分将在链下共识协议和支付过程中被排除;

- More...

更多

Based on these rich on-chain metrics a monitoring Dapp could be built, showcasing
the live status of DOS Network.

基于这些丰富的链上指标，可以构建监控Dapp，并展示DOS网络的实时状态。

Registration system 注册系统

For DOS off-chain instances to join the network, they need to stake and lock some
DOS tokens as security deposit, and to register their deposit address as well as
payment address in the registration contract. They will be registered within at least
one threshold group $Gi$ as groups may overlap with each other. The registration
process of threshold groups are described in the off-chain architecture section
below.

对于DOS链下实例加入网络，他们需要投注并锁定一些DOS token作为保证金，在注册合约中登记他们的
提款地址和支付地址。 它们将在至少1个阈值组$Gi$中进行注册，它们可能会相互重叠。
阈值组的注册过程在下面的 链下结构 一节中介绍。

The security deposit makes the system resistant to sybil attacks and enhances
network security. It also serves as a kind of commitment that the node operator
would contribute bandwidth and computation power to strengthen DOS Network, and
they will be compensated for “mining” rewards as well as earning processing fees.
The lockup period helps stabilize the network to get rid of too frequent registration
and deregistration flips. Any instance with out-of-bound offtime would also be
penalized by forfeiting part of its deposit. Groups with no responses up to certain
limit would be removed from registration system.

保证金使系统能够抵抗女巫攻击(sybil attacks)，提高系统的安全性。
它还能作为节点操作的一种承诺，贡献带宽和计算能力使DOS网络更强大，他们将获得“采矿”奖励并赚取手续费。
锁定期有助于稳定网络，避免过于频繁的注册和注销。
任何超时的情况也将被罚款，没收部分定金。
在一定限度内没有响应的组将从注册系统中删除。

Payment system 支付系统

The payment of a query request goes to the selected threshold group that handles it,
and is distributed among honest members. Payments will be stored in the payment
contract first, as the transfer to node runners doesn’t necessarily happen in real time

- a withdrawal pattern is preferable and node operators are able to check and
withdraw their earnings through a frontend UI or interacting with payment contract
directly.

查询请求的付款将发送到所选择的处理它的“阈值组”里，并在诚实的成员之间分发。
支付将首先存储在支付合约中，因为转移到节点运行器不一定是实时的

—— 退出模式是更好的选择，节点操作者可以检查和提取其收入，通过前端UI或者直接与付款合同进行交互。

DOS token is used in the form of natively supported payment token, as well as the
staking token. However, for blockchains with widely-accepted stablecoins (e.g.
Ethereum), the stablecoin would be a preferable payment token since node runners
won’t be risking for the volatility of crypto prices; the pricing model for the fees will
also be easier to make. We’ll support DOS as payment token first, in the long run,
node runners and token holders will have governance rights to vote for which
stablecoins (DAI/USDC/TUSD/etc.) to be accepted as extra payment tokens.

DOStoken以本地支持的 支付token和抵押token 的形式使用。
然而，对于具有广泛接受的区块链稳定币(例如 Ethereum), 稳定币是一个更好的支付token，
节点运行者不会因价格的波动而面临风险; 收费的定价模式也将更容易制定。
我们将首先支持DOS作为支付token，从长远来看，节点运行者和token持有者将有管理权，投票选择
接受哪些稳定币 (DAI/USDC/TUSD/等)作为额外的支付token。

Different payment schemes will also be supported: pay-per-use will be widely
adopted and suitable for personal developers and light-use Dapps, while discounted
subscription model will be more favorable to heavily dependent applications such as
stablecoins and other decentralized open finance platforms.

不同的支付方案也将得到支持：按次付费将被广泛采用，适合于个人开发者和轻度使用的Dapps，
优惠订阅模式将更有利于高度依赖的应用程序，如稳定币和其他分布式的开放金融平台。

The on-chain system will take modular design pattern into mind that all on-chain
contracts will be upgradable. Since it’s an open and distributed network environment
with different parties have different economic appeals, there’s no simple perfect
model to rule them all. More governance experiments and economic models will be
researched and explored in future.

链上系统将采用模块化设计模式，所有链上合约都是可升级的。
由于它是一个开放的分布式网络环境与不同的群体有不同的经济诉求，没有一个简单的完美模式适合它们。
今后将对更多的治理实验和经济模式进行研究和探索。

---

2.3.2 Part II: Computation Oracle 计算类预言机

Verifiable computation means a client could outsource the computing of some
functions to untrusted third parties with enough computing power. The result, along
with a proof demonstrating the validity and soundness of the result, would be
returned to the client, the client could then execute a verification step instead of
performing the original computation.

可验证的计算意味着客户端可以外包一些计算向不可信的第三方提供功能，并提供足够的计算能力。
结果以及证明结果有效性和可靠性的证明将返回给客户机，客户端随后可以执行验证步骤，而不是执行原始计算。

We could already provide a consensus-based computation oracle solution once part
I is done, as the idea is quite straightforward: For each computation request,
randomly select a group of computing nodes, execute the deterministic computation
task by each group member and send back the result agreed by the majority. This
could serve as our short term plan for the computation oracle service we’d like to
offer to supported blockchains, however, there is a better option we’d like to offer in
the long-term roadmap.

一旦 part I完成，我们就可以提供一个基于共识的计算型预言机，这个想法很简单：
对于每个计算请求，随机选择一组计算节点，每组成员执行确定性计算任务，并将得到多数人同意的结果发送回去。
这可以作为我们想要为支持的区块链提供的计算行预言机服务的短期计划，但是，我们希望在长期路线图中提供更好的选择。

With the successful Byzantium hard fork on October, 2017, Ethereum is now
capable of verifying zkSNARK proofs in smart contracts (see Appendix III). This
enables us to build a zkSNARK-based verifiable computation oracle, where the
execution happens off-chain and verification happens on-chain. The whole process
could be roughly broken down into three phases:

随着2017年10月份拜占庭硬叉的成功，以太坊现在能够在智能合约中验证零知识证明(见 Appendix III)。
这使我们能够构建一个基于零知识证明的可验证计算型预言机，它在链下执行，在链内验证。
整个过程可以大致分为三个阶段：

● Setup phase : For each specific computation, define C is its equivalent
arithmetic circuit, λ is secret random numbers (“toxic-waste” by Zcash) that
must be destroyed after the setup phase. Two public available keys would be
generated by the setup phase: (Pk, V k ) = Setup (λ, C) , where Pk is called
proving key to be used in the off-chain computing phase and V k is called
verification key to be used in on-chain verification phase. This setup phase
only needs to run once for each computation task C , as long as the
computation logic/step doesn’t change, Pk and V k could be reused for
different inputs. A verification contract with hardcoded V k could be deployed
on-chain by now to verify future computation results for different inputs (see
an example contract in Appendix III).

● 设置阶段：对于每个特定的计算，定义 $C$是它的等效算术电路，λ是秘密随机数( Zcash 中所说的有毒废料)
在设置阶段后必须销毁。设置阶段将生成两个公共可用密钥： $(P_k, V_k) = Setup (λ, C)$,
$P_k$ 称为证明公钥，用于链下计算阶段, $V_k$ 称为验证公钥, 用于链上验证阶段。
对于每个计算任务$C$，设置阶段仅需要运行一次，只要计算逻辑/步骤不变,$P_k$和$V_k$可以用于不同的输入。
一个硬编码的验证合约 $V_k$ 被部署在链上, 现在可以通过不同输入的来验证未来的计算结果(见
Appendix III的一个示例合约)。

● Off-chain computation phase : The computation could then be carried on
off-chain with any valid input i . This could further be broken down into two
steps: (i) Performing computation and come up with the result:
o = Compute (C, i) . (ii) Generating a proof for o using proving key Pk :
πproofs = GenerateProof(C, Pk, i, o) . The proof along with the computation
result is then sent back on-chain to the verification contract.

● 链下计算阶段：

然后，可以使用任何有效的输入 $i$ 进行链下计算。这可以进一步分为两个步骤：
(i) 执行计算并得出结果 $o = Compute (C, i)$，
(ii) 使用证明公钥 $P_k$ 生成 $o$的证明：$π_{proofs} = GenerateProof(C, P_k, i, o) $。
然后将证明和计算结果一起发送回链上的验证合约。

● On-chain verification phase : One final step happening on-chain is to check
the validity of computation result o given corresponding input i :
verifier.at(contract address).V erify([πproofs], [i, ... , o]) .

Comparing with consensus-based computation oracle or interactive verification
game between solver and challenger (Truebit), zkSNARK based off-chain
computation only needs to be executed once. There’s no interaction between the
prover and verifier, meaning it is non-interactive. The proof is succinct, meaning its
size is small and independent of the complexity of the computation; the verification
algorithm verifies the validity quickly and it’s also independent of the complexity of
the computation, it only depends on the size of the input. These features make it an
ideal solution to bring unbounded computation power and execution scalability to
blockchains.

● 链上验证阶段：链上发生的最后一步是检查给定相应输入$i$的计算结果$o$的有效性：
  $verifier.at(contract\ address).Verify([π_{proofs}], [i, ... ,  o])$。

与基于共识的计算型的预言机或求解者与挑战者之间的交互验证博弈(Truebit)，基于zkSNARK的链下计算只需要执行一次。
验证者和验证者之间没有交互，这意味着它是非交互的。 证明是简洁的，意味着它很小并且与计算的复杂性无关。
验证算法能快速验证算法的有效性，且不受计算复杂度的影响，它只取决于输入的大小。
这些特性使它成为理想的解决方案，它为区块链带来无限的计算能力和执行的可扩展性。

There’re several tricky points though. First is the toxic waste that’s used in the setup
phase, it must be destroyed and must not be leaked otherwise fake proofs would be
generated to attest computation. Secondly, for the arithmetic circuit generation, a
computation task cannot apply zkSNARK directly before being converted to the right
form called “Rank-1 Constraint System” (R1CS) then finally to “Quadratic Arithmetic
Program” (QAP). Working directly with R1CS or QAP sounds like writing assembly
code by hand, which is error-prone and involves non-trivial work, not friendly for most
developers.

但是有几个棘手的问题。 首先是设置阶段中使用的“有毒废物”，它必须被销毁，不能泄漏，
否则将生成假证明来认证计算。其次，对于算术电路的生成，在转换为名为R1CS(rank-1 constraint system，一阶约束系统)的正确形式然后最终转换为QAP(Quadratic Arithmetic Program)之前，不能直接应用zkSNARK。
直接使用R1CS或QAP听起来像是手写汇编代码，这很容易出错并涉及非常重要的工作，对大多数开发人员来说并不友好。

These problems need to be resolved in order to bring practical zkSNARK-based
verifiable computation to blockchains. A new multiparty computation (MPC)
protocol  that could be scaled to hundreds or even thousands of participants has
recently been proposed by Zcash researchers to fix the trusted setup issue. The
protocol has the property that all of the participants have to be compromised or
dishonest in order to compromise the final parameters. Moreover, now the trustless
setup has been separated into two stages: one big, single “system-wide” trustless
setup called “Powers of Tau” has produced partial zkSNARK parameters to be
reused for all zkSNARK circuits within a given size bound. An application-specific
trustless MPC setup is still needed, but because of the partial parameters produced
by Powers of Tau, the application specific MPC is achievable and much cheaper.

需要解决这些问题，以便将基于zkSNARK的实用可验证计算带入区块链。Zcash研究人员最近提出了
一种新的多方计算（MPC，multiparty computation）协议，可以扩展到数百甚至数千名参与者，以修复可信设置问题。
协议的特性是，所有参与者都必须妥协或不诚实，才能妥协最终的参数。此外，无需信任的设置分为两个阶段：
一个大的，单一的“系统范围”无需信任设置称为 “Powers of Tau” ，它产生了部分 zkSNARK 参数，
可以在给定大小范围内的所有zkSNARK电路中重复使用。仍然需要特定于应用程序的无需信任MPC设置，
但由于“Powers of Tau”产生的部分参数，应用程序特定的MPC是可实现的并且便宜得多。

To address the second issue, we will define and formalize a domain specific
language (DSL) called Zink for verifiable computation, the grammar is similar to
python or javascript, with high-level abstractions such as variables, conditions and
flow control statements, loops, functions, module/file imports, etc. So developers are
able to write off-chain computation code in high-level programming language without
the need of understanding the crazy math under zkSNARK or to deal with low-level
details like R1CS. We’ll also develop a toolchain including an SDK and a front-end
compiler compiling high-level Zink code into low-level R1CS, and connecting to
existing proving system like SCIPR lab’s libsnark as back-end. The toolchain will
also provide a command line tool as well as library code to be integrated into DOS
client software to enable verifiable computations and make them adaptable to
supported chains automatically.

为解决第二个问题，我们需要定义和形式化一个领域特定语言(DSL，domain specific language)
叫 Zink 用于可验证的计算，语法类似于python或javascript，使用高级抽象，如变量、条件和流控制语句、
循环、函数、模块/文件 引入等。
所以开发人员能够用高级编程语言编写链下计算代码，而无需理解zkSNARK的数学或处理像R1CS这样的底层细节。
我们还将开发一个工具链，包括SDK和一个前端编译器，它将高级Zink代码编译成底层R1CS，并作为后端连接
到SCIPR lab的libsnark等现有的证明系统。工具链还将提供命令行工具和库代码，以便集成到DOS客户端软件中，
以实现可验证计算，并使它们自动适配支持的链。

---

2.4 Cross-Chain Interoperability 跨链互操作性

DOS Network opens a door to perform cross-chain interaction between
heterogeneous blockchains. Assuming DOS Network supports data feed oracle
services to both Ethereum blockchain and EOS blockchain, then theoretically, smart
contract on Ethereum is able to trigger cross-chain state changes, flowing through
DOS client nodes, calling into smart contract on EOS. DOS Network thus is acting as
connectors or bridges between supported heterogeneous chains.

DOS网络为异构区块链之间的跨链交互打开了一扇大门。
假设DOS网络支持对以太坊区块链和EOS区块链的数据提供预言机服务，
理论上，以太坊的智能合约能够触发跨链的状态变化，流经DOS客户端节点，再在EOS上调用智能合约。
因此，DOS网络就是充当支持的异构链之间的连接器或桥梁。

A simple application is like exchanging heterogenous crypto asset atomically.
Decentralized exchanges nowadays can only trade homogeneous crypto assets in
the same blockchain, decentralized exchanges e.g. EtherDelta or 0x relayers are
unable to trade against EOS tokens directly.
However, with the help of DOS Network,
it’s achievable by deploying two DEX contracts on Ethereum and EOS
blockchain respectively and defining two cooperative functions to trigger orderbook
and account balance state changes upon calling from the other DEX contract
address through DOS connectors.
This example showcases the potential of DOS Network in cross-chain interoperability.

一个简单的应用程序比如原子级交换异构加密资产。
如今，分布式交易所只能交易同质加密资产在相同的区块链上，这些分布式交易所，
如EtherDelta或0x无法直接与EOS进行交易。
然而，通过DOS网络的帮助，这可以通过在以太坊和EOS区块链上分别部署两个DEX合约来实现，定义两个协作函数，
一个触发定价单，另一个通过DOS连接器从其他 dex 合约地址调用时控制帐户余额状态更改。
这个例子展示了DOS在网络中的跨链互操作的潜力。

The operation and maintenance of any newly supported blockchain’s full node (or
utilizing remote full node services like infura) is to the node operators’ economic
interests and capability, while DOS Network team is responsible for porting and
deploying on-chain system contracts to newly supported chains and releasing
off-chain core client software including protocol update and new adapter support.

任何新支持的区块链的完整节点（或利用infura等远程全节点服务）的运营和维护都
是对节点操作者的经济利益和能力的影响，DOS网络团队负责将链上系统合约移植和部署到新支持的链上，
并发布链下核心客户端软件，包括协议更新和新的适配器支持。

To start up oracle services to newly supported chains we need to go through similar
bootstrapping process as mentioned in section 2.3.1, mainly the one-time group
registration and non-interactive DKG process. Noting that for various supported
blockchains, chain-wide system variables like group size $M$ and number of
registered groups $T$ could be different; the random number $r$ published on different
chains are also different in general.

要为新支持的链启动oracle服务，我们需要经历2.3.1节中提到的类似的引导过程，主要是一次性组注册和非交互式DKG流程。
注意到支持的各种区块链，组大小$M$和注册组数量$T$等全链系统变量可能会不同；在不同的链上发布的随机数$r$通常也是不同的。
